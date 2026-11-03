# How Taptiles / Allin Studio connects GSC, GA4, Google Ads and Meta Ads to the ERP

Source: `wulinhan/taptiles-site` at `71fbef3` (3 Nov 2026). Read-through of `erp/lib/growth-sources.js`, `erp/lib/marketing-analytics.js`, `erp/lib/meta-ads.js`, `erp/lib/metrics-snapshot.js`, `erp/api/cron-metrics.js`, `lib/attribution.js`, `lib/meta-conversions.js`, `lib/google-ads-conversion.js`, `lib/ga4-measurement.js`, `lib/report-purchase.js`, `erp/js/analytics-reconciliation.js`, `erp/MARKETING-ANALYTICS.md`.

The point of this teardown is not to audit the code. It is to name the jobs we had to solve ourselves, because those jobs are the product.

## 1. Architecture at a glance

```
                ┌──────────── Browser (allin-studio.com) ─────────────┐
                │ gtag GA4 + Meta Pixel + src/journey.js events        │
                │ allin_attr cookie: gclid/gbraid/wbraid/fbclid/ttclid │
                │ + UTMs + _ga client id + _fbp/_fbc + landing page    │
                └───────────────┬──────────────────────────────────────┘
                                │ order submit / payment webhook
                                ▼
 ┌──────────── Website API (Vercel) ───────────┐        ┌──── Notion ────┐
 │ lib/attribution.js  -> writes click ids +   │──────▶ │ Orders DB      │
 │   UTMs onto the order (Notion properties)   │        │ (the "ERP")    │
 │ lib/report-purchase.js on confirmed payment │        │ Metrics DB     │
 │   ├─ Meta Conversions API (hashed PII)      │        │ (daily rows)   │
 │   ├─ GA4 Measurement Protocol purchase      │        └───────▲────────┘
 │   ├─ Google Ads offline click conversion    │                │
 │   └─ TikTok Events API                      │                │
 └─────────────────────────────────────────────┘                │
                                                                 │
 ┌──────────── ERP (separate Vercel project, ops.allin-studio.com) ───────┐
 │ GET /api/analytics (Master only, 5 min cache)                          │
 │   GA4 Data API (runReport + v1alpha runFunnelReport)                   │
 │   Search Console searchAnalytics.query                                 │
 │   Google Ads googleAds:search (GAQL)                                   │
 │   Meta Graph /act_.../insights                                         │
 │   Resend metrics, Notion paid orders                                   │
 │ GET /api/cron-metrics (01:30 SGT) -> one row per day into Notion       │
 │ Pages: Conversion funnel, Revenue & growth, GA4-vs-ERP reconciliation  │
 └────────────────────────────────────────────────────────────────────────┘
```

## 2. Per platform

| Platform | Read path | Write path | Auth | Notes |
|---|---|---|---|---|
| GA4 | Data API `runReport` (v1beta) for sessions, channel, revenue; `runFunnelReport` (v1alpha) for `view_item → add_to_cart → begin_checkout → purchase` and cart cohorts at 24h/7d/28d | Measurement Protocol server-side `purchase` with order ref as `transaction_id`, `{ref}-balance` for second payments | Service account JSON, ADC, key file path, or OAuth refresh-token trio. Two different auth code paths exist (google-auth-library in one module, hand-rolled JWT signing in another) | Property pinned to `530895210`; rejects the obsolete property. Hostname regex filter excludes previews. Discloses thresholding/sampling |
| Search Console | `searchAnalytics.query` by date, query (top 100), page (top 25), `dataState: final` | none | Same Google identity, `webmasters.readonly` | Pacific-time dates, multi-day lag, anonymised queries do not sum to totals |
| Google Ads | GAQL over `campaign` with cost, clicks, impressions, conversions, paginated | Offline click conversion upload (gclid / gbraid / wbraid) on confirmed payment, Singapore timestamp format | OAuth refresh token with `adwords` scope; optional developer token and login-customer-id headers | API version env-configurable (`v23`). Conversion value not pulled into the campaign table, so ROAS left blank by design |
| Meta Ads | Graph `insights` at campaign level: spend, impressions, clicks, link clicks, reach, actions, action_values, purchase_roas | Conversions API: Lead, InitiateCheckout, Purchase etc, SHA-256 hashed email/phone/name, fbp/fbc, event_id dedup with Pixel | Long-lived `META_ADS_READ_TOKEN` (read) and `META_PIXEL_TOKEN` (CAPI). Account id and Pixel id hard-coded | Picks `omni_purchase` first to match Ads Manager. Purchases explicitly never added to GA4 revenue |
| ERP (Notion Orders) | Paid orders in range, deposits vs balances, first landing, latest campaign tags | Attribution properties added to the DB automatically, copy kept in page body in case a column is missing | Notion integration token | Acts as the source of truth for money |

## 3. The ten jobs we actually had to solve

These are the reusable lessons. Each one maps to a product capability later in the PRDs.

1. **Credential plumbing, twice.** Around 15 environment variables across two Vercel projects (`GROWTH_GOOGLE_*`, `GA4_*`, `GSC_*`, `META_*`, `GOOGLE_ADS_*`). OAuth refresh tokens were minted by hand in OAuth Playground. Rotating a scope means "reauthorize, paste into Vercel, redeploy".
2. **The assistant connection and the business-system connection are separate silos.** The code says it plainly: *"Meta is connected in chat; the ERP needs its own server-side read token."* The Claude chat connector to Meta could not be reused by our own systems, and our ERP connection could not be reused by chat. Same data, two setups, two sets of numbers.
3. **Error translation.** `growthFailure()` maps raw Google errors into nine human fixes (API disabled, missing scope, OAuth needs reauthorizing, property access, quota, and so on). This is a large share of the real support burden of any data connector, and it is invisible in demos.
4. **Measurement truth rules.** `MARKETING-ANALYTICS.md` is essentially a rulebook: do not add Meta purchases to GA4 revenue; no click-to-session rate across systems; Search Console is property-wide, Meta is account-wide; filters apply to GA4 only; drop-off is "not observed", not "abandoned"; windows are cumulative, not additive. These rules are exactly what an LLM breaks when you hand it four raw APIs.
5. **Identity of a sale across systems.** A single order id (`TAP-123`, `TAP-123-balance`) threads through GA4 `transaction_id`, Google Ads `order_id`, Meta `event_id`, and the ERP. Without this, reconciliation is impossible.
6. **Click-id capture at the edge.** A first-party cookie stores gclid, gbraid, wbraid, fbclid, ttclid, UTMs, `_ga`, `_fbp`, `_fbc` and the first landing page, then copies them onto the order record.
7. **Closing the loop (write-back).** Server-side CAPI, GA4 Measurement Protocol, and Google Ads offline conversions fire from the payment webhook, so ad platforms optimise on real paid orders, including deposits and balances.
8. **Reconciliation.** GA4 purchase ids are matched against ERP orders and flagged: missing refs, duplicates, abandoned, part-paid, non-positive revenue.
9. **History beyond API retention.** A daily cron writes one row per day per source into Notion so trends survive Search Console's 16 month window, account changes, and API outages. Sources fail soft with a note.
10. **Scope and safety.** Read-only by construction ("No campaigns can be written by this endpoint"), Master-only access, no raw provider errors or tokens in logs, internal staff traffic excluded from analytics.

## 4. What it cost

- Roughly two weeks of focused iteration (files first appear 20 to 29 Oct 2026) across at least five PRs (#40, #41, #55, #62 plus follow-ups such as the balance transaction id fix).
- About 1,500 lines of integration-specific code before UI, plus a 170 line rulebook and two verification scripts.
- Ongoing cost: API version bumps (Google Ads `v23`, Graph `v23.0` read vs `v21.0` CAPI), token expiry, and every new client account needing its own ids hard-coded or configured.

## 5. What is still missing in our own build

Worth noting, because these gaps are likely shared by prospective customers:

- No natural-language access. The ERP shows dashboards; asking "why did CPA jump on Tuesday" still means opening four tools or pasting into chat by hand.
- No multi-account or multi-client support (ids hard-coded). An agency or a second brand would need a fork.
- TikTok has a server-side Events API sender (`lib/tiktok-events.js`, fired from the same purchase fan-out) but no reporting adapter; organic social has neither.
- Google Ads conversion value is not pulled, so Google ROAS stays blank.
- No alerting (spend spikes, tracking breakage such as Pixel events dropping to zero, CAPI match quality falling).
- Two Google auth implementations and duplicated configuration.

## 6. Takeaway

The read layer (pulling numbers from four APIs) was the easy 30%. The hard 70% was: credentials that both humans and machines can use, error recovery, a semantic rulebook that stops wrong maths, a shared order id, and the write-back loop to the ad platforms. That 70% is where a product can win, especially now that the platforms themselves are shipping free official read-only connectors (see `02-market-research.md`).
