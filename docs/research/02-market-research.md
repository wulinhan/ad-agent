# Market research: connecting marketing data to AI assistants

Prices were checked on 3 Nov 2026, on the vendor's own pricing page unless noted. R = read, W = write. "All four" = Search Console + GA4 + Google Ads + Meta Ads.

## 1. Direct competitors (marketing data into Claude / ChatGPT / others)

| Product | Sources | Assistants | R/W | Joins sources or own revenue data? | Entry price and meter | Built for |
|---|---|---|---|---|---|---|
| Windsor.ai | 350+, all four, Shopify, HubSpot | Claude, ChatGPT, Gemini, Perplexity | R, W on ad platforms (you confirm each change) | Warehouse export; no join inside the connector | Free (1 source, 1 account). Basic $23/mo (3 sources). Standard $118 (7). Meter: sources + accounts | SMB to agency |
| Supermetrics | All four, CRM, Shopify, Klaviyo | Claude, ChatGPT, Copilot, Gemini Ent., n8n | R, some W (new campaigns created paused) | No | Starter $55/mo (3 sources × 3 accounts, 1 user). MCP needs Data API tier. Meter: sources × accounts × users + AI credits | Mid-market, agencies |
| Adzviser | ~70 incl. all four, Shopify | ChatGPT, Claude, Gemini, Perplexity, Manus | R | Bundled warehouse capped by rows | Starter $48.99 (3 sources), Growth $148.99 (7) | SMB, freelancers |
| Porter Metrics | 26+ | Claude, ChatGPT app | R only, by design | No | Free (3 accounts, 30 days history). Solo $14.99. Teams $39.99. Meter: accounts (third-party source) | SMB, freelancers |
| Coupler.io | 400+ | Claude, ChatGPT, Gemini, Perplexity, Cursor, Copilot | R + builds data flows | **Yes, SQL joins across datasets** | Starter $32/mo (3 accounts). Active $132 (15). Meter: accounts + AI quota | SMB to mid |
| Dataslayer | Major ad + analytics | Claude, ChatGPT, Gemini, Mistral, Copilot | R | No | MCP from €29/mo (yearly). Bundled MCP capped at 100 messages/mo | SMB |
| Funnel.io | 600+, normalised | Claude, ChatGPT, Cursor, Gemini, Codex | R | Yes, data hub + custom metric definitions | From $300/mo | Mid-market, enterprise |
| Improvado | 187 incl. CRM | Claude, ChatGPT, Cursor | R + W + monitoring | Implied | Sales-led | Enterprise |
| Whatagraph | Agency sources | Claude + ChatGPT directories | R, W on reports | Yes ("blends") | From €699/mo | Agencies |
| Two Minute Reports | 30+ incl. all four | Claude, ChatGPT, Manus | R | In its reports, not clearly via MCP | $97/mo (3 accounts per source) | Agencies |
| AgencyAnalytics | 80+ | Claude, ChatGPT directories | R, W on its own app | Uploaded files; one client per connection | $20 per client per month | Agencies |
| Swydo | 34+ | Claude, ChatGPT | R | No | €69/mo incl. 10 accounts | Agencies |
| Pipeboard | Meta, Google Ads, TikTok, LinkedIn; GA4/GSC free | Any MCP client | **R + W** (budgets, duplication, bulk) | No | Free (2 accounts, 30 calls/week). Pro $29.90. Meter: ad-account slots | SMB to agency |
| Adspirer | 6+ ad platforms, GA4, Klaviyo | ChatGPT, Claude, Cursor, Codex | R + W | No | Free 15 tasks. Plus $40. Meter: tasks | SMB to agency |
| GoMarble | 5 ad platforms | Claude, ChatGPT | R; W with approval from $399 | No | $139 / $399 / $799. Meter: ad spend + accounts + credits | Ecommerce, agencies |
| Markifact | 50+ | Claude, ChatGPT, Cursor, Manus | R + W with approval | Via workflows | Pro $79/mo. Meter: credits | SMB to agency |
| 1ClickReport | All four + Stripe, Shopify | Claude, ChatGPT | R, W (created paused) | Claims "true revenue", unexplained | $25 / $99 / $199 (vendor blog) | SMB, agencies |
| CData Connect AI | Hundreds | Claude, ChatGPT, others | R + W with approval | Saved cross-source views (Growth) | $99/mo for 1 source; extra sources $59 to $299 each | IT, mid-market |
| Composio / Pipedream / Zapier MCP / Make MCP | Generic app toolkits | Any | R + W | No | Usage-metered (per call, per task, per end user) | Developers, ops |

### Free, official, platform-owned servers (the commoditising force)

| Server | Scope | Catch |
|---|---|---|
| Meta Ads MCP (official) | Full R + W across campaigns, audiences, catalogs, A/B tests, benchmarks. Open beta April 2026, opened to any app 16 Jul 2026. Exposes about 100 tools in our own Claude session | Meta only. No GSC/GA4/Google/revenue context |
| Google Analytics MCP (official, open source) | GA4 read, optional annotations | Self-hosted; needs Google credentials. 3.6k GitHub stars |
| Google Ads MCP (official, open source) | Read + narrow write (assets, labels; no budgets or pausing) | Self-hosted; needs Cloud project + developer token. 1.1k stars |
| Search Console | **No official Google server.** Community: AminForou/mcp-gsc (2.1k stars) | Local install, own OAuth client |

## 2. Adjacent competitors (attribution and ecommerce analytics with AI)

Several prices here are third-party snapshots because vendor pages blocked automated access. Treat them as indicative.

| Vendor | Price and meter | MCP / assistant access | Server-side conversions | Segment |
|---|---|---|---|---|
| Triple Whale (Moby) | By GMV, 12 month contract. Foundation ~$219/mo (<$250K GMV) to ~$2,529 ($15M to $20M) (competitor snapshot) | **Yes**, official read-only hosted MCP, OAuth `moby:read` | Yes (Sonar Send) | Shopify DTC |
| Polar Analytics | % of GMV, unpublished | **Yes**, Claude + ChatGPT, plus "Headless MCP" with a 400+ metric semantic layer | Yes, Meta + Google ("Advertising Signals") | Shopify DTC |
| **Cometly** | By ad spend, mostly custom quote | **Yes**, Claude Desktop, ChatGPT, Cursor | Yes: Meta CAPI, Google enhanced + offline, LinkedIn, TikTok, Microsoft | SMB, agencies, **lead-gen** (closest overlap to us) |
| Lifetimely (AMP) | By orders, $149 to $999/mo | "Lifetimely MCP" listed, few details | Not found | Shopify SMB |
| Northbeam | ~$1,500/mo+ (third-party) | No official MCP | Partial (Meta only) | Mid/enterprise DTC |
| Hyros | Custom, median ~$25K/yr reported | Unofficial only | Not verified | Info products, high-ticket |
| Elevar | By orders, $225 to $1,250/mo | None | Yes, core product | Shopify |
| Littledata | $0.35/order or from $159/mo | No | Yes | Shopify |
| Stape | By requests, free to ~$200/mo | **Yes**, free open-source Stape + GTM MCPs | Yes (server-side GTM) | Any stack, SMB to agency |
| Klaviyo, HubSpot Breeze | Platform pricing | **Yes**, official MCPs | n/a | Ecommerce / B2B |
| Shopify Sidekick | Free | Store ops MCPs, not cross-source analytics | No | Shopify |

**Read:** in Shopify DTC, "attribution data inside Claude/ChatGPT" is already sold by Triple Whale, Polar, Lifetimely and Cometly. Outside Shopify (custom stacks, Notion/Airtable/Sheets ERPs, service and lead-gen businesses, non-US payment rails such as HitPay), nobody joins ad + search + analytics data to real orders, reconciles them, and enforces cross-system maths rules.

## 3. How the three assistants accept connectors (Nov 2026)

| Assistant | Mechanism | Plan gates | Write safety |
|---|---|---|---|
| Claude | Remote MCP custom connectors, plus a curated connector directory | Free gets **one** custom connector. Team/Enterprise: only Owners add connectors, members then connect | Per-tool permission prompts in client |
| ChatGPT | Apps directory (submit via OpenAI platform) and Developer mode for any MCP server | Developer mode not on Free; Business: admins deploy custom apps. Read-only vs write on Plus/Pro unclear, verify | Write actions confirmed by default |
| Grok | Remote MCP in API (native SDK, Responses API, voice). Consumer connectors launched 6 May 2026 incl. custom remote MCP | Possibly paid plans only (unverified) | **No `require_approval`**: the server must enforce its own confirmation |

None of roughly 25 vendors checked lists Grok as supported.

## 4. Platform constraints (these shape the product more than competitors do)

Items marked **[verify]** came from secondary sources or could not be confirmed on the platform's own page.

### Google Ads: the big one
- **Proxy ban, Ads Developer Policy update 31 Aug 2026.** Hosted MCP servers or wrappers that *solely* re-expose Google Ads API capabilities are prohibited, and each advertiser or agency is expected to use its own Google Cloud project. Open-source tools the user runs with their own credentials are allowed. A hosted service may need a "Developer Secondary Interface Review" (reported to expect SOC 2 Type II level controls). **[verify with Google before building]** whether a value-added analytics product that joins Ads data with orders falls outside "solely re-exposes".
- **Developer tokens retired 9 Sep 2026.** Access now attaches to the Google Cloud project: Test (test accounts), Explorer (2,880 production ops/day, automated review), Basic (15,000 ops/day, brand verification required), Standard (unlimited, ~10 business day manual audit). Projects inactive for 90 days lose access.
- **Offline click conversions moved to the Data Manager API.** New adopters calling `UploadClickConversions` in the Google Ads API since 15 Jun 2026 get `CUSTOMER_NOT_ALLOWLISTED_FOR_THIS_FEATURE` unless they had qualifying use Dec 2025 to May 2026. **This affects our own `lib/google-ads-conversion.js`, which still calls `googleads.googleapis.com/.../:uploadClickConversions`.**

### Google OAuth, GA4, Search Console
- `adwords`, `analytics.readonly`, `webmasters.readonly` appear to be *sensitive*, not *restricted*: app verification (typically 3 to 5 business days), no annual CASA assessment. **[verify]**
- GA4 Data API: 200K core tokens per property per day, and **14K tokens per project per property per hour**, the limit a multi-tenant product hits first. 10 concurrent requests per property.
- Search Console: 25,000 rows per request, 50K rows per day per search type, **16 months of retention**. Daily snapshots are the only way to keep longer history.

### Meta
- Access tiers renamed: Limited (was Standard) and Full (was Advanced). Full needs App Review, Business Verification, 500+ successful calls in 15 days, under 15% errors.
- Insights rate limit per ad account per hour: 600 + 400 × active ads on development, 190,000 + 400 × active ads on Full.
- System user tokens are documented as non-expiring; at least one developer report says otherwise. Always check `expires_at`. **[verify]**
- CAPI: events at most 7 days old (one stale event rejects the batch), dedup on `event_id` + `event_name` within 48 hours.

### MCP authorization
- Current spec (2026-07-28) recommends **Client ID Metadata Documents (CIMD)** and deprecates Dynamic Client Registration (kept for backward compatibility).
- Claude supports both; ChatGPT needs Developer Mode on paid plans with user OAuth and sends no custom headers; Grok accepts custom remote MCP by URL on paid plans, OAuth for custom servers unclear **[verify]**, and no client-side approval for writes.

### Platform-owned AI is moving up the stack
- Meta Ads AI Connectors (`mcp.facebook.com/ads`), free, read + write, no developer credentials.
- Google Analytics MCP and Google Ads MCP, open source, self-hosted.
- Google **Ask Advisor** spanning Ads + Analytics + Merchant Center, limited beta, wider rollout planned by end of 2026.

## 5. Demand signals

| Signal | Figure | Source |
|---|---|---|
| SMBs using AI marketing tools | 54% (Constant Contact Q1 2026, n=1,500+); 45% use AI for trend/data analysis | chainstoreage.com |
| US SMB AI adoption | 87% in 2026, up from 26% in 2023 (Constant Contact) | news.constantcontact.com |
| Marketers using AI for analysing results | 67% of social and 73% of retail-media AI users (Digiday+, Q1 2026) | digiday.com |
| Meta SMBs using its AI ad tools | 9M+ (Q2 2026) | earnings coverage |
| US digital ad agencies | ~100K businesses (IBISWorld 2026) | ibisworld.com |
| Meta advertisers | "10M+" widely cited, last official figure is old | adexchanger.com |
| Google Ads advertisers | Not disclosed | n/a |

## 6. Sources

Full URL list in `sources.md`.
