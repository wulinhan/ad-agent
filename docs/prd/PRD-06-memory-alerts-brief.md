# PRD-06 · Memory, alerts and the weekly brief

| | |
|---|---|
| Status | Draft v0.1 |
| Phase | 1 (snapshots + weekly brief) → 2 (alerts, custom brief) |
| Depends on | PRD-01, PRD-02, PRD-03 |

## 1. Problem

Two things break the "just ask your assistant" model:
1. **History disappears or shifts.** Search Console keeps 16 months; platforms restate recent data; accounts get reorganised. Taptiles solved this with a daily cron writing one row per day into Notion.
2. **Owners do not ask.** Assistants are reactive. Busy owners need the answer to arrive: a weekly brief, and an alert when something breaks (tracking dies, spend spikes, a token expires).

No hosted marketing MCP we found sends proactive briefings.

## 2. Goals

- G1. Daily immutable snapshots per workspace, per system, so trends and "what did we know then" are always available.
- G2. A weekly brief, written in plain language, with the truth-layer guarantees (PRD-02), delivered where the owner reads.
- G3. Alerts for the few things that cost money when missed.

## 3. Functional requirements

### 3.1 Snapshots
| ID | Requirement | Priority |
|---|---|---|
| MB-1 | Daily job per workspace after each system's data settles (configurable; default 01:30 workspace time, re-snapshot day −3 for GSC/GA4 restatements) | P0 |
| MB-2 | Columns at minimum as Taptiles `metrics-snapshot.js`: orders, booked revenue, receipts, GA4 sessions/purchases/revenue, Ads spend/clicks/conversions, Meta spend/purchases/value/clicks/impressions/ROAS, GSC clicks/impressions, plus notes on sources unavailable | P0 |
| MB-3 | Sources fail soft: a missing source writes a note, never blocks the row | P0 |
| MB-4 | Optional export of snapshots to the customer's Notion, Google Sheets or BigQuery, so the customer owns their history | P1 |
| MB-5 | Retention: 25 months on Starter, unlimited on Growth and above | P1 |

### 3.2 Weekly brief
| ID | Requirement | Priority |
|---|---|---|
| MB-10 | Default Monday 08:00 workspace time, covering the latest complete Monday to Sunday vs the previous week and trailing 28 days (Taptiles default) | P0 |
| MB-11 | Sections: headline (cash MER, orders, receipts); each channel side by side; search movers (queries/pages); funnel; reconciliation status; signal health (if PRD-04 on); 3 suggested actions with evidence | P0 |
| MB-12 | Generated with the same tools and guardrails as assistant answers; every number carries provenance in a footnote | P0 |
| MB-13 | Delivery: email (P0); Slack, WhatsApp, Telegram (P1); a link that opens the brief as a prompt in Claude, ChatGPT or Grok for follow-up (P1) | P0/P1 |
| MB-14 | Owner can edit tone and sections once; agency can set a template across clients | P1 |

### 3.3 Alerts
| ID | Trigger | Priority |
|---|---|---|
| MB-20 | Tracking break: GA4 purchases or Meta Pixel purchases drop to zero for 24 hours while orders continue | P0 (phase 2) |
| MB-21 | Spend spike: platform daily spend > 1.5× trailing 7 day average with no matching order rise | P0 (phase 2) |
| MB-22 | Connection about to fail: token expiring, scope removed, API disabled, quota near limit | P0 (phase 1, from PRD-01) |
| MB-23 | Reconciliation gap widens beyond workspace threshold | P1 |
| MB-24 | Search: a top-20 page loses > 40% of clicks week on week | P1 |

Every alert includes the evidence and one suggested next step; snooze and threshold controls per alert.

## 4. Metrics

- Weekly brief open rate (≥ 60%) and follow-up rate (clicked "ask about this", ≥ 20%).
- Alerts acted upon or acknowledged (≥ 50%); false-positive feedback (< 20%).

## 5. Open questions

- Is the brief the real product for most owners, with chat as the power feature? Pilot data should decide which we lead with on the website.
- Should we write the brief in the owner's own voice when they forward it to staff or clients (agency use case)?
