# Positioning

Working name: **Truecount**. Alternatives worth testing: Groundline, Paidtrue, Countwell. (Avoid "Tally", which collides with Tally Forms.)

Method: April Dunford's sequence (alternatives, then unique attributes, then value, then who cares most, then category), because the category we sit in decides who we get compared to.

## 1. Competitive alternatives (what customers do today)

1. **Paste screenshots and CSVs into ChatGPT or Claude.** Free, slow, error-prone, no history.
2. **Connect each platform's own connector** (Meta Ads AI Connector, Google's GA4 MCP). Free, but each one only knows itself, so the assistant happily adds Meta purchases to GA4 revenue.
3. **A connector reseller** (Windsor, Adzviser, Porter, Supermetrics). $15 to $150/mo, priced by source or account, raw numbers with no rules, no order data.
4. **A Shopify attribution suite** (Triple Whale, Polar). Not available or not worth it off Shopify; priced on GMV.
5. **Build it yourself**, as we did at Taptiles. Two weeks for a technical founder, then ongoing upkeep.
6. **Do nothing**, and keep trusting whichever dashboard claims the most credit.

## 2. Unique attributes (true today in our internal build, to be productised)

| Attribute | Proof point |
|---|---|
| Joins marketing data to **real paid orders from your own system**, not just Shopify | Taptiles ERP joins Notion orders to GA4, Google Ads, Meta and GSC |
| **Guardrailed maths**: refuses to add numbers across attribution systems, labels scope (account-wide vs property-wide), discloses lag and thresholding | `MARKETING-ANALYTICS.md` rulebook, enforced in code |
| **Reconciliation**: shows which GA4 purchases match an order, which are missing, duplicated, part-paid | `analytics-reconciliation.js` |
| **Closes the loop**: sends paid orders back to Meta, Google and GA4 with one shared order id | `report-purchase.js` fan-out |
| **One connection, every assistant**: Claude, ChatGPT, Grok, plus your own dashboards via API | Gap seen in our own code: chat and ERP needed separate tokens |
| **Fixes its own plumbing**: turns provider errors into one-line fixes | `growthFailure()` |
| **Remembers**: daily snapshots beyond platform retention | `cron-metrics` into Notion |

## 3. Value (what the attributes let the customer do)

- **Know which marketing made money**, in the currency that hits the bank, in a sentence.
- **Trust the answer**, because every number says where it came from and what it excludes.
- **Spend less to get the same sales**, because the ad platforms optimise on real orders instead of guesses.
- **Stop maintaining plumbing**: tokens, scopes, API versions, Google's 2026 policy changes.

## 4. Who cares most (beachhead)

**Owner-operated businesses doing roughly $300K to $10M a year, spending $2K to $50K a month across Google and Meta, whose orders live outside Shopify.** Typical shapes:

- Custom-product and made-to-order ecommerce (like Taptiles: deposits, balances, quotes).
- Service businesses with bookings or enquiries (catering, clinics, studios, workshops, education).
- B2B lead-gen where "revenue" lives in a CRM or a spreadsheet.
- Asia-Pacific businesses on local payment rails (HitPay, Stripe, PayNow) and Notion/Airtable/Sheets as the ERP.

Secondary: fractional marketers and small agencies (2 to 15 people) who run these accounts and want one setup per client.

**Not for (yet):** Shopify DTC brands already on Triple Whale/Polar; enterprises with a warehouse and a data team.

## 5. Market category

Do **not** position as a "marketing data connector" or "MCP server". That category is priced per source and is collapsing towards free.

Position as: **"the revenue truth layer for your AI assistant"**. Short form for the website: *Marketing answers your AI can't get wrong.*

This puts us in a category where the comparison is "your own spreadsheet and a lot of guessing", not "Windsor at $23".

## 6. Positioning statement

> For owner-run businesses that sell outside Shopify and advertise on Google and Meta, **Truecount** is the revenue truth layer for your AI assistant. It joins Search Console, Analytics, Google Ads and Meta Ads to the orders in your own system, so Claude, ChatGPT or Grok can tell you which marketing actually made money, without adding up numbers that don't belong together. Unlike connector tools that pipe raw platform data into chat, Truecount reconciles every sale, shows where each number came from, and sends your real orders back to the ad platforms so they learn from what you actually sold.

## 7. Messaging pillars

| Pillar | Headline | Supporting points | Proof |
|---|---|---|---|
| **Truth** | Your AI, minus the made-up maths | Never sums across attribution systems. Every answer cites source, scope and date range. Flags lag, thresholding, sampling | Rulebook enforced in tools; "provenance footer" on every answer |
| **Money** | Ad numbers, matched to real sales | Connect Notion, Airtable, Sheets, Stripe, HitPay, WooCommerce. Deposits and balances handled. Reconciliation report | Taptiles: GA4 purchases matched to `TAP-123` and `TAP-123-balance` orders |
| **Loop** | Teach the ad platforms what you actually sold | Meta CAPI, GA4 Measurement Protocol, Google Data Manager API, one shared order id, dedup built in | Taptiles purchase fan-out |
| **Anywhere** | One connection. Claude, ChatGPT, Grok, and your own dashboards | Same numbers in every assistant and in your ERP via API or a daily sheet/Notion snapshot | MCP + REST + snapshot |
| **Calm** | It fixes its own plumbing | Plain-English fixes for expired tokens, missing scopes, disabled APIs. Weekly brief arrives on its own | Error translator; proactive brief |

## 8. Proof we can show in the first month

- A Taptiles case study: before (four tools, two weeks of build, numbers that disagreed) vs after (one question in Claude, reconciled answer).
- A side-by-side demo: the same question asked through raw official connectors (assistant double counts) and through Truecount (assistant refuses, explains, gives the right figure).

## 9. Objections and answers

| Objection | Answer |
|---|---|
| "Meta and Google give me connectors for free." | Use them. They each only see themselves. Ask them "which channel made me money last month" and watch them disagree. We join them to your orders and stop the double counting. |
| "Windsor is $23." | It sends raw numbers. We sell a correct answer about money, plus the write-back that improves your ads. Different job. |
| "I'm not technical." | Setup is sign-in buttons plus one guided Google Cloud step, or we do it for you on a call. |
| "Is my data safe?" | Read-only by default. Writes need your approval on our side, even in assistants that don't ask. Tokens stay in our vault, never in chat. |
| "What about Google's new rules?" | We built for them: customer-owned Cloud project option, Data Manager API for conversions, and no raw Ads API pass-through. |

## 10. Words to use and avoid

- **Use:** real sales, matched, where this number came from, account-wide, your orders, send back, approve.
- **Avoid:** "single source of truth" (overused and untrue across attribution systems), "AI-powered insights", "blended ROAS" (that is precisely the maths we refuse to do silently), "unlimited", "MCP" in consumer copy.

## 11. Pricing posture (detail in PRD-00)

Flat price **per business**, all four platforms included. This undercuts the per-source meter that pushes SMBs from $23 to $118 at Windsor or $49 to $149 at Adzviser, and it reads as fair to agencies who stack businesses.
