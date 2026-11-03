# PRD-04 · Signal loop: conversion write-back and signal health

| | |
|---|---|
| Status | Draft v0.1 |
| Phase | 2 |
| Depends on | PRD-01, PRD-03 |

## 1. Problem

Ad platforms optimise on the conversions they can see. Browser pixels miss a growing share (ad blockers, iOS, consent, offline payments, deposits paid later by bank transfer). Taptiles sends confirmed payments server-side to Meta (Conversions API), GA4 (Measurement Protocol) and Google Ads (offline click conversions), all keyed on the same order id. That is the part of our build most directly linked to ad performance.

Google changed the ground in 2026: offline click conversion upload moved to the **Data Manager API**, and new adopters of `UploadClickConversions` in the Google Ads API get `CUSTOMER_NOT_ALLOWLISTED_FOR_THIS_FEATURE` since 15 Jun 2026. Taptiles' own `lib/google-ads-conversion.js` still uses the old endpoint and may be silently skipped. Many DIY setups are in the same position.

## 2. Goals

- G1. Turn on server-side conversions for Meta, GA4 and Google from the order source, without code.
- G2. Deduplicate correctly with browser events.
- G3. Show **signal health** in plain words and in the assistant: "Meta saw 31 of your 45 paid orders this week; match quality 6.8/10; 2 events rejected as older than 7 days."

## 3. Non-goals

- Full server-side tag manager hosting (Stape's business). We send purchase-grade business events from orders, not every page view.

## 4. Functional requirements

| ID | Requirement | Priority |
|---|---|---|
| SL-1 | Trigger on order payment events from PRD-03 sources (webhook where available, otherwise polling with idempotency) | P0 |
| SL-2 | **Meta CAPI**: Purchase (and Lead for lead-gen), hashed em/ph/fn/ln/external_id, fbp/fbc, client IP/UA when captured, `event_id = purchase_{ref}_{phase}` matching the browser Pixel; reject-safe batching so one stale event (> 7 days) does not drop the batch | P0 |
| SL-3 | **GA4 Measurement Protocol** purchase with `transaction_id = {ref}` or `{ref}-balance`, client id from `_ga` when present | P0 |
| SL-4 | **Google Data Manager API** for click conversions (gclid/gbraid/wbraid) and enhanced conversions for leads; account allowlist check at setup with a clear message if Google has not enabled it | P0 |
| SL-5 | Value rules: cash collected per phase by default; option for full order value on first payment; currency from order | P0 |
| SL-6 | Never block or fail the customer's payment webhook; queue with retries and dead-letter list | P0 |
| SL-7 | Signal health per platform: events sent, accepted, rejected (with reason), dedup rate, Meta Event Match Quality, share of paid orders that reached each platform | P0 |
| SL-8 | Assistant tool `signal_health` and a weekly brief section; alert when acceptance drops > 30% week on week or a platform receives zero events for 48 hours | P0 |
| SL-9 | TikTok Events API (Taptiles already sends `CompletePayment`) | P1 |
| SL-10 | Backfill: optional replay of the last 7 days (Meta limit) / 90 days (Google limits vary) on first enable, with a dry run | P1 |
| SL-11 | Consent: send only events with lawful basis per workspace setting; LDU flags for US state privacy | P0 |

## 5. UX

- Toggle per platform on a "Send real sales back" page, each with a dry-run preview of the next 5 events (hashed fields masked).
- Health card per platform, green/amber/red with the one-line reason and fix.

## 6. Metrics

- % of Pro workspaces with at least one platform enabled (≥ 70%).
- Median share of paid orders reaching Meta (target uplift vs pixel-only baseline, measured per workspace).
- Customer-reported CPA change after 30 days (self-reported in pilot, not a promise in marketing).

## 7. Risks

- **Google allowlisting** may exclude new customers from click conversion upload entirely. Mitigation: support enhanced conversions for web/leads paths and GA4-linked conversions as fallbacks; be explicit in onboarding.
- **Double counting** if the customer also runs Elevar/Stape/another CAPI. Mitigation: detect existing server events via Meta dataset diagnostics before enabling; require confirmation.
- **Legal**: hashed PII transfer. Mitigation: DPA, consent gating, regional processing.

## 8. Open questions

- Should Signal loop be a separate SKU priced on ad spend, since its value scales with spend?
- Do we migrate Taptiles first to validate the Data Manager API path (recommended: yes, it is also a live bug risk for us)?
