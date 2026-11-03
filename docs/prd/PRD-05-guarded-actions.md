# PRD-05 · Guarded actions

| | |
|---|---|
| Status | Draft v0.1 |
| Phase | 3 |
| Depends on | PRD-01, PRD-02, PRD-06 |

## 1. Problem

Once an owner trusts the answer ("campaign X spent $400 and produced no paid orders in 14 days"), the next request is "so pause it". Write-capable tools exist (Meta's official MCP, Pipeboard, Adspirer, GoMarble), but guardrails are thin: "created paused" or a client-side approval prompt at best. Grok's remote MCP has **no client-side approval** at all. No vendor we found publishes budget caps or rollback.

Taptiles deliberately stayed read-only. This PRD defines how we allow a few high-value writes without becoming the reason someone's ad account burns money.

## 2. Goals

- G1. A small set of reversible actions tied to evidence from the truth layer.
- G2. **Server-side approval** that works the same in every assistant, including those that do not ask.
- G3. Every change is logged, attributable, and one-click reversible.

## 3. Non-goals

- Creating new campaigns, creatives or audiences (the platforms' own connectors do this; we do not compete there).
- Autonomous optimisation without a human.

## 4. Action set (v1)

| Action | Platforms | Reversible by |
|---|---|---|
| Pause / enable campaign, ad set, ad group, ad | Google Ads, Meta | Enable / pause |
| Change daily budget within ±30% (workspace configurable) | Google Ads, Meta | Restore previous value |
| Add negative keywords from GSC/Ads search terms | Google Ads | Remove |
| Annotate GA4 (e.g. "Paused campaign X") | GA4 | Delete annotation |

## 5. Requirements

| ID | Requirement | Priority |
|---|---|---|
| GA-1 | Two-step protocol: `propose_change` returns a change id, plain-English summary, evidence (from PRD-02 with provenance), expected effect, and rollback; `apply_change` requires a confirmation code delivered **outside the assistant** (email, Slack, WhatsApp/Telegram, or the web app) unless the workspace opts into in-chat confirmation for clients that support it | P0 |
| GA-2 | Policy engine per workspace: allowed actions, max budget change %, max daily total spend change, blackout hours, who can approve | P0 |
| GA-3 | Hard limits independent of policy: no budget increase > 2× in 24 hours, no changes to more than 10 entities per approval | P0 |
| GA-4 | Immutable audit log: who proposed (assistant + user), who approved, before/after values, provider response | P0 |
| GA-5 | One-click rollback within 7 days; auto-suggest rollback if spend or CPA moves adversely beyond threshold in 48 hours | P1 |
| GA-6 | Viewer role and API keys without write scope can never propose or apply | P0 |
| GA-7 | Google Ads writes only through value-added tools, never a raw mutate pass-through (PRD-01 §6) | P0 |

## 6. Metrics

- Approved / proposed ratio (signals proposal quality).
- Rollbacks within 7 days (< 10% of applied changes).
- Zero incidents of changes applied without approval.

## 7. Risks

- **Liability**: a mistaken pause during a sale. Mitigation: blackout windows, clear terms, approvals, rollback.
- **Policy**: Meta `ads_management` requires Full access and review. Plan review timing with PRD-01.

## 8. Open question

- Is "suggest only, with a deep link to make the change in Ads Manager" enough for most SMBs? It avoids most risk and may be the better phase 3 default.
