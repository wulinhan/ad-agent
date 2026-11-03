# PRD-03 · Revenue link: order sources, identity and reconciliation

| | |
|---|---|
| Status | Draft v0.1 |
| Phase | 1 (Notion, Google Sheets, Stripe) → 2 (HitPay, Airtable, WooCommerce, HubSpot, CSV upload, Postgres) |
| Depends on | PRD-01, PRD-02 |
| Feeds | PRD-04, PRD-06 |

## 1. Problem

Marketing platforms report what they *claim*. The business banks what it *received*. For non-Shopify businesses, received money sits in a Notion database, a spreadsheet, Stripe, HitPay, a CRM. Nobody in the connector market joins these for an SMB, and no assistant connector reconciles them.

Taptiles shows what it takes: a first-party attribution cookie, click ids and UTMs written onto each order, one order id threaded through GA4, Google Ads and Meta (with `-balance` for second payments), and a reconciliation view that flags mismatches.

## 2. Goals

- G1. Connect an order source in under 10 minutes, with a mapping wizard for messy data.
- G2. Every paid order carries its marketing context (first landing, latest campaign, click ids) whenever the site can capture it.
- G3. Reconcile GA4 purchase events and platform-claimed conversions against recorded orders, and explain gaps.
- G4. Give PRD-02 a trustworthy `orders` system: cash basis, deposits and balances distinguished, refunds netted.

## 3. Non-goals

- Replacing the customer's ERP, invoicing or payment system.
- Payment settlement or refund auditing (Taptiles explicitly does not do this either).

## 4. User stories

1. As Ada, I connect my Notion orders database, tell Truecount which column is the order id, amount, paid date and status, and see last month's revenue match what I expect.
2. As Ada, I install one snippet (or a Google Tag Manager template) so new orders carry their click ids and UTMs.
3. As Mei, I see a list: "GA4 recorded 42 purchases, your system has 45 paid orders. 3 have no GA4 event (likely ad blockers or direct bank transfer). 1 GA4 purchase has no matching order (test order?)."
4. As Ada, I ask Claude "which campaigns brought orders over $500 last quarter" and get an answer based on my orders, not Meta's claims.

## 5. Functional requirements

### 5.1 Source connectors
| ID | Source | Method | Phase |
|---|---|---|---|
| RL-1 | Notion database | Notion OAuth, pick DB, map properties | 1 |
| RL-2 | Google Sheets | Google OAuth (`spreadsheets.readonly`), pick sheet/tab, map columns | 1 |
| RL-3 | Stripe | Stripe Connect read-only; charges, payment intents, invoices, refunds; metadata for order id and UTMs | 1 |
| RL-4 | HitPay | API key, payment requests + webhooks (SEA focus) | 2 |
| RL-5 | Airtable | OAuth, pick base/table | 2 |
| RL-6 | WooCommerce | REST keys | 2 |
| RL-7 | HubSpot deals | OAuth; closed-won amount and date as "revenue" for lead-gen | 2 |
| RL-8 | CSV upload / Postgres read replica | Manual or scheduled | 2 |

### 5.2 Mapping wizard
| ID | Requirement | Priority |
|---|---|---|
| RL-10 | Auto-suggest mappings for: order id, created at, paid at (deposit / balance / full), amount, currency, status, customer email/phone (hashed on ingest for matching only), product/category, and attribution fields | P0 |
| RL-11 | Support multi-payment orders (deposit + balance) as Taptiles does, with phase-specific transaction ids `{ref}` and `{ref}-balance` | P0 |
| RL-12 | Preview: last 30 days totals by day, with "does this look right?" confirmation before saving | P0 |
| RL-13 | Status mapping (paid, part-paid, abandoned, refunded, test) with a "test order" marker excluded from reporting | P0 |
| RL-14 | Data quality score: % orders with valid id, amount, paid date, attribution | P1 |

### 5.3 Attribution capture
| ID | Requirement | Priority |
|---|---|---|
| RL-20 | Lightweight first-party script and GTM template that stores gclid, gbraid, wbraid, fbclid, ttclid, UTMs, `_ga` client id, `_fbp`, `_fbc`, first landing page in a first-party cookie (port of Taptiles `lib/attribution.js`) | P0 |
| RL-21 | Helpers to attach that payload to the order: hidden form fields, Stripe Checkout metadata, Notion property write-back, webhook enrichment | P0 |
| RL-22 | Consent-aware: honours Consent Mode / CMP signals; no capture without consent where required | P0 |
| RL-23 | First landing is never overwritten; latest click id replaces earlier ones (Taptiles semantics) | P0 |
| RL-24 | No raw PII stored in the attribution payload; email/phone only hashed and only when PRD-04 needs them | P0 |

### 5.4 Reconciliation
| ID | Requirement | Priority |
|---|---|---|
| RL-30 | Match GA4 `purchase` transaction ids to orders using the workspace's id pattern (configurable regex, default learned from data) | P0 |
| RL-31 | Flags: no matching order, order with no GA4 event, duplicate transaction id, abandoned order with purchase event, part-paid, non-positive revenue, currency mismatch | P0 |
| RL-32 | Platform claims vs recorded: Meta purchases and Google Ads conversions shown next to orders with matching click ids, as "claimed" vs "matched" (never summed, PRD-02 R1) | P1 |
| RL-33 | Reconciliation summary exposed to assistants via `revenue_truth` and to humans on a reconciliation page | P0 |
| RL-34 | Weekly reconciliation section in the brief (PRD-06) | P1 |

## 6. Data model (simplified)

```
Order(id, workspace, external_ref, created_at, status, currency, total,
      payments[Payment(phase, amount, paid_at, transaction_id)],
      attribution(first_landing, utm_*, click_ids{gclid,gbraid,wbraid,fbclid,ttclid},
                  ga_client_id, fbp, fbc),
      customer_hash(email_sha256?, phone_sha256?), product_category, is_test)
```

## 7. Metrics

- % workspaces with an order source connected in week 1 (≥ 50%).
- % paid orders with any click id or UTM after snippet install (target ≥ 60% for paid traffic).
- Reconciliation match rate GA4 ↔ orders (report, not target; median baseline in pilot).

## 8. Risks

- **Messy sources** (free-text amounts, mixed currencies, no ids). Mitigation: wizard, data quality score, concierge.
- **Privacy**: storing click ids and hashed contact data. Mitigation: data processing agreement, regional storage (SG/EU/US), retention limits, deletion API.
- **Snippet adoption** is low without developer help. Mitigation: GTM template, Wix/Squarespace/Webflow guides, concierge.

## 9. Open questions

- Should lead-gen businesses (no online payment) use "qualified lead" as the money proxy, with a value per lead? Sofnade-style catering enquiries suggest yes.
- Which SEA payment rails after HitPay: Xendit, 2C2P, Omise?
