# PRD-00 · Truecount product overview

| | |
|---|---|
| Status | Draft v0.1 for discussion |
| Owner | Linhan Wu |
| Date | 3 Nov 2026 |
| Related | `research/01`, `research/02`, `strategy/03`, `strategy/04`, PRD-01 to PRD-06 |

## 1. Problem

Owner-run businesses advertise on Google and Meta, get organic traffic from Search, and track their site in GA4. Their money lives somewhere else: Notion, Airtable, a spreadsheet, Stripe, HitPay, WooCommerce, a CRM. Each marketing platform counts sales its own way, and none sees the real orders.

AI assistants are now where these owners go to think. But when you connect raw platform data to Claude, ChatGPT or Grok, the assistant adds Meta purchases to GA4 revenue, compares Search Console clicks with GA4 sessions as if they were the same thing, and has no idea which orders were actually paid.

At Taptiles we solved this for ourselves in about two weeks and 1,500 lines of code. Most businesses cannot.

## 2. Vision

Any owner can ask their own AI assistant "which marketing made us money this week, and what should I change", and get an answer that is reconciled against real orders, honest about its limits, and the same in every assistant.

## 3. Personas

| Persona | Description | Top job |
|---|---|---|
| **Owner-operator ("Ada")** | Runs a $300K to $10M business, does marketing alone or with one freelancer, uses ChatGPT or Claude daily | "Tell me if my ad spend is working, in money, without me opening four dashboards" |
| **Fractional marketer / small agency ("Ravi")** | Manages 3 to 15 such businesses | "Give each client a correct weekly answer without building reports by hand" |
| **Ops / finance helper ("Mei")** | Keeps the order system tidy, chases payments | "Make the marketing numbers match the money we actually received" |

## 4. Jobs to be done

1. When I check how the week went, I want one honest answer about marketing and money, so I can decide where to put next week's budget.
2. When numbers disagree between platforms, I want to know why, so I stop arguing with dashboards.
3. When I run ads, I want the platforms to learn from real paid orders, so my cost per sale drops.
4. When something breaks (tracking, token, spend spike), I want to be told before it costs me.

## 5. Product shape

```
  Claude · ChatGPT · Grok · (Gemini, Cursor later)        Owner's own dashboards / ERP
               │  remote MCP (OAuth)                              │ REST API · daily snapshot
               ▼                                                  ▼
      ┌──────────────────────────── Truecount ─────────────────────────────┐
      │ PRD-02 Truth layer: metrics, rules, provenance, approval gate      │
      │ PRD-06 Memory: daily snapshots, alerts, weekly brief               │
      │ PRD-05 Actions: guarded write operations                           │
      ├────────────────────────────────────────────────────────────────────┤
      │ PRD-01 Connection vault: OAuth, customer-owned Cloud project mode, │
      │        error translator, health checks                             │
      │ PRD-03 Revenue link: order sources, identity, reconciliation       │
      │ PRD-04 Signal loop: Meta CAPI, GA4 MP, Google Data Manager API     │
      └──────┬─────────┬────────────┬────────────┬───────────────┬─────────┘
            GSC       GA4      Google Ads     Meta Ads    Orders (Notion, Airtable,
                                                           Sheets, Stripe, HitPay, Woo…)
```

## 6. PRD map and phasing

| PRD | Capability | Phase |
|---|---|---|
| PRD-01 | Connection vault and multi-assistant gateway | 1 (MVP) |
| PRD-02 | Truth layer: metric semantics, guardrails, provenance | 1 (MVP) |
| PRD-03 | Revenue link: order sources and reconciliation | 1 (MVP, two sources) → 2 (more) |
| PRD-06 | Memory, alerts and weekly brief | 1 (snapshots + brief) → 2 (alerts) |
| PRD-04 | Signal loop: conversion write-back and signal health | 2 |
| PRD-05 | Guarded actions (pause, budget, negatives) | 3 |

**Phase 1, MVP (target 10 weeks):** GSC, GA4, Google Ads, Meta Ads read; Notion + Google Sheets + Stripe order sources; truth layer tools; works in Claude, ChatGPT, Grok; daily snapshot; weekly brief by email. Pilot with Taptiles + 20 businesses.

**Phase 2 (weeks 11 to 20):** HitPay, Airtable, WooCommerce, HubSpot sources; signal loop write-back with health scores; alerts; agency workspace.

**Phase 3:** guarded actions; TikTok and Microsoft Ads; Shopify as a source; white-label briefs.

## 7. Out of scope (for now)

- Building our own dashboard product beyond setup, health and reconciliation screens. The assistant is the interface.
- Multi-touch attribution modelling or media mix models. We report each system's own attribution honestly and reconcile to orders; we do not invent a new model.
- Creative generation, ad copy, audience building.
- Becoming a data warehouse. We keep snapshots and caches, and export to the customer's tools.

## 8. Pricing (proposal to test)

Meter: **per business** (one brand with its own ad accounts and order source). All four platforms always included. Any number of assistants.

| Plan | Price (monthly / yearly) | Includes |
|---|---|---|
| Free | $0 | 1 business, GA4 + GSC read, 30 days history, 1 assistant |
| Starter | $29 / $24 | All four platforms, all assistants, 13 months snapshots, weekly brief |
| Growth | $79 / $66 | + 1 order source, reconciliation, alerts, REST API, 3 users |
| Pro | $149 / $124 | + signal loop write-back with health monitoring, 3 order sources, guarded actions (phase 3), priority support |
| Agency | $25 per business / month, min 5 | Growth features per business, client workspaces, shared brief template |
| Concierge setup | $390 one-off | We set up Cloud project, tracking ids, order mapping on a call |

Rationale: an SMB needing four sources pays $118 at Windsor Standard, $149 at Adzviser Growth, $55+ at Supermetrics (3 source cap). $79 with orders and reconciliation is clearly better value; $29 entry undercuts most connector resellers while the Free tier competes with the official free servers.

**Price test to run:** half the pilot sees Growth at $79, half at $99. Also test an ad-spend-linked Pro for write-back (for example 1% of ad spend capped at $299) since its value scales with spend.

## 9. Success metrics

| Type | Metric | Phase 1 target |
|---|---|---|
| North star | Weekly businesses that received a reconciled answer (assistant query or brief that used order data) | 15 of 20 pilot businesses by week 8 |
| Activation | Time from sign-up to first correct answer in an assistant | Median under 20 minutes (self-serve), under 1 day (concierge) |
| Activation | % of businesses connecting an order source in week 1 | ≥ 50% |
| Trust | Answers flagged "wrong" by users per 100 answers | < 3 |
| Trust | Guardrail interventions that users agreed with (thumbs up on "I won't add these because…") | ≥ 80% |
| Reliability | Connections healthy at any time | ≥ 97% |
| Commercial | Pilot → paid conversion | ≥ 40% |
| Commercial | Logo churn month 2 to 4 | < 5% / month |

## 10. Key risks

| Risk | Mitigation | Owner |
|---|---|---|
| Google Ads proxy policy blocks hosted access | Written guidance from Google before MVP build; customer-owned Cloud project mode; value-added tools only (PRD-01 §6) | Founder |
| Google sensitive-scope OAuth verification delays launch | Start verification in week 1; pilot under testing-mode user cap | Eng |
| Meta Full access needs 500 calls / 15 days and Business Verification | Start App Review early, dogfood with Taptiles + agency accounts to build call volume | Eng |
| LLMs ignore guardrail notes and still sum numbers | Make wrong maths impossible at the tool layer, not just in prose (PRD-02) | Eng |
| Order sources are messy (no consistent order id, mixed currencies) | Mapping wizard + "confidence" score + concierge setup | Product |
| Grok lacks approval prompts | Server-side approval gate for all writes (PRD-05) | Eng |

## 11. Open questions for Linhan

1. Is the first customer group really "non-Shopify SMBs", or the Singapore/SEA SMB network you already reach through Allin and Sofnade-type clients? These overlap but the channel differs.
2. Should Taptiles itself migrate onto Truecount (dogfooding, removes ~1,500 lines), or stay as the reference implementation?
3. Self-serve first, or concierge first? (`strategy/03` §7 argues for testing concierge.)
4. Appetite for SOC 2 Type II within 12 months, given Google's secondary-interface review may expect it?
5. Brand: is Truecount a standalone brand or part of the Allin Creatives family?
