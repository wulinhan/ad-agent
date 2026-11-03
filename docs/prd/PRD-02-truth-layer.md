# PRD-02 · Truth layer: metric semantics, guardrails and provenance

| | |
|---|---|
| Status | Draft v0.1 |
| Phase | 1 (MVP) |
| Depends on | PRD-01 |
| Feeds | PRD-03, 05, 06 |

## 1. Problem

Hand an LLM four raw marketing APIs and it will confidently produce wrong answers:
- Adds Meta "purchases" to GA4 purchases to get "total sales".
- Divides GA4 sessions by Search Console clicks as a "click-to-visit rate".
- Applies a device filter to Meta and Search Console numbers that cannot be filtered that way.
- Treats funnel drop-off as "abandoned carts".
- Compares a Pacific-time Search Console week with a Singapore-time GA4 week.

At Taptiles we wrote a 170 line rulebook (`MARKETING-ANALYTICS.md`) and enforced it in code. That rulebook is the most valuable thing we built, and no SMB-priced competitor ships one. This PRD turns it into the product's core.

## 2. Goals

- G1. Make the common wrong answers **impossible at the tool layer**, not merely discouraged in a prompt.
- G2. Every number returned carries **provenance**: source system, attribution model, scope (account-wide / property-wide / filtered), date range and timezone, freshness, and known caveats.
- G3. Keep the tool surface small and intention-shaped so it works well in Claude, ChatGPT and Grok alike.

## 3. Non-goals

- A custom multi-touch attribution model.
- A general SQL interface over the customer's data.

## 4. Core concepts

### 4.1 Measurement systems
Every metric belongs to exactly one **system**: `gsc`, `ga4`, `google_ads`, `meta`, `orders` (PRD-03). Each system has a declared:
- **scope**: what filters it honours (for example Meta: campaign only; GSC: query/page/country/device; GA4: full);
- **attribution**: GA4 data-driven/last click per property setting, Google Ads account setting, Meta account setting (for example 7-day click, 1-day view), Orders = cash recorded;
- **clock**: timezone and lag (GSC Pacific time, finalised after ~2 to 3 days; GA4 24 to 48 hours processing);
- **currency** (from the connection, never assumed).

### 4.2 Metric catalogue
A versioned catalogue (YAML) defines each metric: id, system, formula, unit, valid dimensions, and **compatibility rules**. Starting set, from the Taptiles build:

| Metric | System | Notes |
|---|---|---|
| impressions, clicks, ctr, position | gsc | property-wide; anonymised queries do not sum to totals |
| sessions, users, purchases, purchase_revenue, funnel steps | ga4 | hostname filtered to production; thresholding disclosed |
| spend, impressions, clicks, conversions, conversion_value, cpa, roas | google_ads | account currency; conversion action set declared |
| spend, impressions, reach, clicks, link_clicks, purchases (omni first), purchase_value, roas | meta | account currency and timezone |
| orders, booked_revenue, receipts, deposits, balances, refunds | orders | from PRD-03, cash basis |

### 4.3 Guardrail rules (initial set)
| Rule | Behaviour |
|---|---|
| **R1 No cross-system sums** | Any request to add, subtract or average a metric across systems is rejected with an explanation and a correct alternative ("show side by side" or "use orders as the money total") |
| **R2 No cross-system ratios** unless whitelisted | Example allowed: `orders.receipts / (google_ads.spend + meta.spend)` = **blended MER on cash**, labelled as such and only when currencies match. Example rejected: `ga4.sessions / gsc.clicks` |
| **R3 Scope honesty** | If a filter does not apply to a system, the tool returns the unfiltered figure with `scope: account_wide` and a visible note, never a silently mis-filtered number |
| **R4 Clock alignment** | Ranges are converted to each system's clock; tools report actual dates used; incomplete days flagged `still_processing` |
| **R5 Language rules** | Funnel drop-off is "not observed at next step", not "abandoned"; cohorts within window flagged "still converting" |
| **R6 Currency** | No arithmetic across currencies without an explicit FX table and label |
| **R7 Small numbers** | Changes on fewer than N events (default 20) are flagged "too few to call a trend" |
| **R8 Self-reported vs observed** | Platform-claimed conversions are labelled "claimed by Meta/Google"; orders are labelled "recorded in your system" |

### 4.4 Provenance envelope
Every tool result includes:
```json
{
  "value": 1234.5,
  "metric": "meta.purchase_value",
  "system": "meta",
  "attribution": "7d_click_1d_view (account setting)",
  "scope": "account_wide",
  "range": {"start":"2026-10-27","end":"2026-11-02","tz":"Asia/Singapore"},
  "freshness": "complete",
  "currency": "SGD",
  "caveats": ["Claimed by Meta. Do not add to GA4 revenue or order totals."],
  "source_ref": "tc://ws_123/meta/act_188.../insights?run=abc"
}
```
Assistants are instructed (via tool descriptions and a short server `instructions` block) to cite the system and scope in answers. A one-line "where these numbers came from" footer is provided ready to paste.

## 5. Tool surface (MCP and REST)

Intention-shaped, few tools. Each validates against the catalogue before calling providers.

| Tool | Purpose |
|---|---|
| `get_business_context` | Workspaces, connected systems, currencies, timezones, attribution settings, health. Assistants call this first |
| `summarise_period` | The default "how did last week go": per-system headline metrics, period-over-period, orders totals, cash MER, flagged anomalies |
| `compare_channels` | Side-by-side per system and per channel; never summed across systems |
| `explain_change` | Decomposes a change in a metric into drivers within one system (campaign, query, page, device), then cross-checks against orders |
| `search_performance` | GSC queries/pages with CTR and position, opportunity flags (high impressions, low CTR, position 4 to 15) |
| `funnel` | GA4 closed funnel and cart cohorts (24h/7d/28d), with Taptiles semantics |
| `campaigns` | Google Ads and Meta campaign tables, sortable inside each platform |
| `revenue_truth` | Orders by first landing / latest campaign / click id; reconciliation summary (PRD-03) |
| `metric_definition` | Returns the catalogue entry and rules for any metric, so the assistant can explain itself |

No generic "run arbitrary query" tool (also required by Google Ads policy, PRD-01 §6).

## 6. Behaviour across assistants

- Tool descriptions are written to be model-agnostic and tested with an eval suite of 60 questions (from real Taptiles and pilot questions) across Claude, ChatGPT and Grok.
- Eval scoring: correct number, correct refusal, provenance cited, no forbidden arithmetic in the final prose.
- Because a model can still add numbers in its own prose, `summarise_period` and `compare_channels` return the correct totals pre-computed and explicitly list the totals that must not be computed.

## 7. Requirements

| ID | Requirement | Priority |
|---|---|---|
| TL-1 | Metric catalogue v1 with all metrics in §4.2 and rules R1 to R8 | P0 |
| TL-2 | Provenance envelope on every numeric result | P0 |
| TL-3 | Tools in §5 except `explain_change` | P0 |
| TL-4 | `explain_change` with driver decomposition for GA4, Ads, Meta | P1 |
| TL-5 | Cross-assistant eval suite run on every release; release blocked below 90% pass | P0 |
| TL-6 | Workspace-level custom definitions (e.g. which Ads conversion actions count, which Meta action type is "purchase") | P1 |
| TL-7 | Disclosure of GA4 thresholding/sampling and row caps | P0 |
| TL-8 | User feedback hook in answers ("Was this right?") logged against the tool call | P1 |

## 8. Metrics

- Eval pass rate per assistant (≥ 90%).
- User-flagged wrong answers per 100 (< 3).
- Share of answers citing system and scope (≥ 95% in sampled transcripts where users opt in).

## 9. Risks

- **Too strict feels unhelpful.** Mitigation: every refusal offers the nearest correct answer in the same response.
- **Catalogue drift** when platforms rename fields (Meta action types). Mitigation: contract tests against recorded fixtures, monthly review.

## 10. Open questions

- Should "blended MER on cash" be the default headline? It is the one cross-system number that is honest, and owners love a single number. Risk: it hides channel differences.
- Do we expose the catalogue publicly as an open standard ("marketing metric contract for LLMs") to build credibility and inbound?
