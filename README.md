# Truecount (working name): research, positioning and PRDs

Product idea: let owner-run businesses connect Google Search Console, GA4, Google Ads, Meta Ads **and their own order system** to the AI assistant they already use (Claude, ChatGPT, Grok), with answers reconciled against real sales.

Origin: the Taptiles / Allin Studio ERP build in `wulinhan/taptiles-site` (`erp/lib/growth-sources.js`, `erp/lib/marketing-analytics.js`, `lib/report-purchase.js` and friends).

## Read in this order

| # | Doc | What it answers |
|---|---|---|
| 1 | [research/01-taptiles-integration-teardown.md](docs/research/01-taptiles-integration-teardown.md) | How we connected the four platforms to our ERP, and the ten jobs we had to solve |
| 2 | [research/02-market-research.md](docs/research/02-market-research.md) | ~30 competitors with pricing, assistant support, and 2026 platform constraints |
| 3 | [strategy/03-gap-and-opportunity.md](docs/strategy/03-gap-and-opportunity.md) | Where the gap is, why now, threats, options, kill criteria |
| 4 | [strategy/04-positioning.md](docs/strategy/04-positioning.md) | Positioning, messaging pillars, objections, pricing posture |
| 5 | [prd/PRD-00-product-overview.md](docs/prd/PRD-00-product-overview.md) | Vision, personas, scope, phasing, pricing, success metrics |
| 6 | [prd/PRD-01-connection-vault-and-gateway.md](docs/prd/PRD-01-connection-vault-and-gateway.md) | Connections, multi-assistant MCP gateway, Google Ads policy design |
| 7 | [prd/PRD-02-truth-layer.md](docs/prd/PRD-02-truth-layer.md) | Metric catalogue, guardrails, provenance, tool surface |
| 8 | [prd/PRD-03-revenue-link.md](docs/prd/PRD-03-revenue-link.md) | Order sources, attribution capture, reconciliation |
| 9 | [prd/PRD-04-signal-loop.md](docs/prd/PRD-04-signal-loop.md) | Server-side conversions (Meta CAPI, GA4 MP, Google Data Manager API) and signal health |
| 10 | [prd/PRD-05-guarded-actions.md](docs/prd/PRD-05-guarded-actions.md) | Reversible writes with server-side approval |
| 11 | [prd/PRD-06-memory-alerts-brief.md](docs/prd/PRD-06-memory-alerts-brief.md) | Daily snapshots, alerts, weekly brief |

Sources: [research/sources.md](docs/research/sources.md). Prices and policies checked 3 Nov 2026; items marked **[verify]** need confirmation from the platform.
