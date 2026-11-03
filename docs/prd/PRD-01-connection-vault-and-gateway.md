# PRD-01 · Connection vault and multi-assistant gateway

| | |
|---|---|
| Status | Draft v0.1 |
| Phase | 1 (MVP) |
| Depends on | none |
| Feeds | PRD-02, 03, 04, 05, 06 |

## 1. Problem

At Taptiles, connecting four platforms meant about 15 environment variables across two deployments, refresh tokens minted by hand in OAuth Playground, two separate Google auth implementations, and a Meta connection in Claude chat that our ERP could not reuse. Every scope change meant "reauthorize, paste into Vercel, redeploy". When something failed, Google returned errors only an engineer could read.

For a customer, the connection step is where most connector products lose people. It also carries our largest policy risk (Google Ads).

## 2. Goals

- G1. A non-technical owner connects GSC, GA4, Google Ads and Meta Ads in under 15 minutes.
- G2. **One connection serves every consumer**: Claude, ChatGPT, Grok, our REST API, and snapshot exports. No re-connecting per assistant.
- G3. Comply with Google's 2026 Ads API policy and Meta's access tiers by design.
- G4. Every failure is turned into a one-line fix the owner can act on.

## 3. Non-goals

- Supporting every ad platform in phase 1 (TikTok, Microsoft, LinkedIn later).
- Acting as a general-purpose integration platform (that is Composio/Pipedream territory).

## 4. User stories

1. As Ada, I click "Connect Google", approve once, and pick my GA4 property, Search Console site and Ads account from lists, so I never type an id.
2. As Ada, I add Truecount to Claude, ChatGPT and Grok by pasting one URL and signing in, and all three see the same business.
3. As Ravi, I connect 8 client businesses under one login and switch between them in the assistant by name.
4. As Ada, when my Meta token expires, I get an email and an in-assistant message saying exactly which button to press.
5. As Mei, I can see which connections are healthy and when each last returned data.

## 5. Functional requirements

### 5.1 Workspace model
| ID | Requirement | Priority |
|---|---|---|
| CV-1 | Account → Workspaces (one per business) → Connections. Agency accounts hold many workspaces | P0 |
| CV-2 | Each workspace stores: business name, reporting timezone, reporting currency, production hostnames (for GA4 hostname filter), internal-traffic exclusions (emails, IPs) | P0 |
| CV-3 | Roles: Owner, Editor, Viewer. Viewer cannot see or trigger writes | P0 |

### 5.2 Google connection
| ID | Requirement | Priority |
|---|---|---|
| CV-10 | Single Google OAuth consent requesting `analytics.readonly`, `webmasters.readonly`, `adwords` (and `datamanager` only when PRD-04 is enabled). Incremental consent: ask for Ads and Data Manager only when the user turns those on | P0 |
| CV-11 | After consent, list accessible GA4 properties, GSC properties (domain and URL-prefix) and Ads customer ids (incl. MCC children); user picks; we store ids | P0 |
| CV-12 | Validate the GA4 property against the site's measurement id when a hostname is given (Taptiles lesson: an obsolete property was in use) | P1 |
| CV-13 | **Two modes for Google Ads access** (see §6): (a) Truecount Cloud project, (b) **customer-owned Cloud project** with a guided setup that creates the project, enables APIs, and registers it, with Truecount operating under the customer's project | P0 |
| CV-14 | Detect and explain Explorer vs Basic vs Standard access limits for the project in use | P1 |

### 5.3 Meta connection
| ID | Requirement | Priority |
|---|---|---|
| CV-20 | Facebook Login for Business with `ads_read` (and `ads_management` only when PRD-05 actions are enabled); list ad accounts, Pixels/datasets, Pages | P0 |
| CV-21 | Prefer a system-user token issued into the customer's Business Manager where possible; read `expires_at` on every token and schedule renewal reminders 14 and 3 days ahead | P0 |
| CV-22 | Store account currency and timezone with the connection; never assume SGD (Taptiles defaulted to SGD) | P0 |

### 5.4 Order sources
Defined in PRD-03; use the same vault, health and error model.

### 5.5 Assistant gateway
| ID | Requirement | Priority |
|---|---|---|
| CV-30 | One remote MCP endpoint per account (Streamable HTTP), workspace chosen by tool argument or by a "current business" setting | P0 |
| CV-31 | OAuth 2.1 authorisation server supporting **CIMD** (preferred by the 2026-07-28 MCP spec) and DCR (backward compatibility for clients that still use it). Allowlist Claude callbacks on both `claude.ai` and `claude.com` | P0 |
| CV-32 | Tested and documented setup for Claude (custom connector), ChatGPT (developer mode app, then directory listing), Grok (remote MCP URL). Note plan gates in onboarding (Claude Free = 1 custom connector; ChatGPT developer mode not on Free) | P0 |
| CV-33 | Submit to Claude connector directory and ChatGPT app directory once stable | P1 |
| CV-34 | REST API with the same tools and the same truth layer (PRD-02) for the customer's own systems, using workspace API keys | P1 |
| CV-35 | Per-assistant session audit: which assistant called which tool, when, with what arguments (no PII payloads) | P0 |

### 5.6 Health and error translation
| ID | Requirement | Priority |
|---|---|---|
| CV-40 | Classify every provider failure into a fixed code set, extending Taptiles' `growthFailure()`: `api_disabled`, `missing_scope`, `reauthorize`, `authentication_failed`, `property_access`, `not_found`, `quota_exceeded`, `invalid_report`, `token_expiring`, `policy_blocked`, `provider_error` | P0 |
| CV-41 | Each code has an authored plain-English fix with a deep link (for example straight to the "enable API" page for the right Cloud project). Never return raw provider messages, URLs, tokens or key material to the assistant or logs | P0 |
| CV-42 | Hourly lightweight health probe per connection; status page per workspace showing last success, last error, quota used | P0 |
| CV-43 | Tools return a structured `unavailable` result with the fix text rather than failing, so the assistant can tell the user what to do (Taptiles fails soft per source) | P0 |

## 6. Google Ads policy design (must be resolved before build)

Google's Ads Developer Policy update of 31 Aug 2026 prohibits hosted MCP servers or wrappers that *solely* re-expose Google Ads API capabilities and expects each advertiser or agency to use its own Cloud project. Developer tokens were retired on 9 Sep 2026; access levels attach to the Cloud project.

Design stance:
1. **No raw pass-through.** We will not expose a generic "run any GAQL" tool. Google Ads data is only reachable through value-added tools (joined to orders, guardrailed, reconciled; PRD-02/03).
2. **Customer-owned project mode is first-class**, not an afterthought. Onboarding automates as much as Google allows (project creation, API enablement, OAuth client) and falls back to a 5 minute guided checklist or the concierge call.
3. **Ask Google in writing** whether mode (a) is acceptable for a value-added analytics product, and what the Developer Secondary Interface Review needs. Record the answer in this PRD.
4. **Fallback:** if hosted Ads access is not permitted, ship GA4 + GSC + Meta + orders hosted, and offer a small open-source local Ads bridge the customer runs (or the official Google Ads MCP) that our truth layer can call.

## 7. UX outline

1. Sign up → "Name your business" → timezone and currency auto-detected from GA4 once connected.
2. "Connect Google" → consent → pick property, site, Ads account → green ticks.
3. "Connect Meta" → Facebook Login → pick ad account and Pixel.
4. "Connect your orders" (PRD-03).
5. "Add to your assistant" → tabs for Claude, ChatGPT, Grok with copy-paste URL and 3 screenshots each → "Try asking: *How did last week go?*"
6. Health page always one click away; red items show the fix text and a "Fix it" button.

## 8. Technical notes

- Secrets in a KMS-backed vault, envelope encrypted per workspace; tokens never reach the assistant.
- Per-project quota accounting for GA4 (the 14K tokens per project per property per hour limit is the first wall a multi-tenant product hits). Cache identical reports for 5 minutes as Taptiles does.
- API versions pinned per provider with a monthly upgrade job and contract tests (Taptiles drifted to Graph v21 for CAPI and v23 for reads).
- Reuse from Taptiles: `growthFailure()` codes, `googleToken()` service account + refresh token flows, Meta row parsing (`metaAction`, `omni_purchase` preference).

## 9. Metrics

- Connection completion rate per platform (target ≥ 85% Google, ≥ 80% Meta).
- Median minutes to all-four-connected (< 15).
- % of errors resolved by the user without support (≥ 70%).
- Connections healthy (≥ 97%).

## 10. Open questions

- Do we need SOC 2 Type II before the Google secondary-interface review, or can we start in customer-owned mode only?
- Will Grok's consumer app support OAuth for custom MCP servers, or do we need a token-in-URL fallback with tight scoping?
- Do we let agencies bring their own Meta app (higher rate limits under their Full access)?
