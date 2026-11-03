# Gap and opportunity

## 1. The market in one paragraph

Getting a single platform's numbers into Claude or ChatGPT is now a solved and nearly free problem. Meta ships an official write-capable MCP for free, Google open-sourced GA4 and Google Ads servers, and Google's Ask Advisor will answer cross-Google questions natively by the end of 2026. Twenty-plus connector vendors (Windsor, Supermetrics, Adzviser, Porter, Coupler and others) resell the same read layer, priced per source or per account, aimed mostly at agencies. Shopify brands can already get attribution data into assistants through Triple Whale, Polar and Lifetimely. So "connect your ads to ChatGPT" is not a business on its own any more.

## 2. Where the gap is

Our own build (see `01-taptiles-integration-teardown.md`) showed that the read layer was the easy 30%. Mapping the hard 70% against the market:

| Job we had to solve at Taptiles | Who does it today | Gap |
|---|---|---|
| One set of credentials usable by both the AI assistant and our own systems | Nobody. Chat connectors and server tokens are separate silos | **Open** |
| Plain-English recovery from auth, scope, quota and API-disabled errors | Partially, inside each vendor's UI | Open for assistant-first UX |
| Measurement truth rules (no cross-system sums, scope labels, lag, thresholding) | Funnel and Polar have semantic layers at $300+/mo or % of GMV. SMB tools hand raw numbers to the LLM | **Open below $100/mo** |
| Joining ad/search/analytics data to real orders from a custom stack (Notion, Airtable, Sheets, Stripe, HitPay, Xero, WooCommerce, Postgres) | Shopify-only tools; Coupler via DIY SQL; Funnel at $300+ | **Open for non-Shopify SMBs** |
| Shared order id across GA4, Ads, Meta and ERP, plus reconciliation | Elevar/Littledata (Shopify). Nobody exposes reconciliation to an assistant | **Open** |
| Server-side conversion write-back (CAPI, GA4 MP, Google offline) | Stape, Elevar, Littledata, Cometly, Triple Whale | Crowded, but **Google's move to the Data Manager API resets the field for new adopters** |
| History beyond API retention (GSC 16 months) | Warehouse add-ons | Partly open |
| Works in Claude, ChatGPT **and Grok**, with server-side approval for writes | No vendor lists Grok. Grok has no client-side approval | **Open** |
| Proactive weekly brief instead of waiting to be asked | Not found in hosted MCP products | **Open** |

## 3. The opportunity, stated plainly

> Small businesses that do not run on Shopify have no trustworthy way to ask their AI assistant "which marketing actually made us money last week", because their money lives in their own order system and their marketing lives in four platforms that each count differently.

The product is a **revenue truth layer** between those platforms, the business's own order data, and whichever assistant they already use. It earns its price by being right (guardrails, reconciliation, provenance) and by closing the loop (sending real paid orders back to the ad platforms), not by fetching numbers.

## 4. Why now

1. Assistants all speak MCP, and Grok joined in May 2026. A single server reaches all three.
2. The platforms' own free read connectors raise demand for cross-source answers while making single-source tools worthless. They do the education for us.
3. Google's 2026 changes (proxy ban, developer token retirement, Data Manager API) break a lot of existing DIY setups and some vendors. Businesses will need a compliant path.
4. SMB AI adoption crossed the majority line (54% use AI marketing tools, 45% for data analysis).

## 5. Threats and how seriously to take them

| Threat | Severity | Response |
|---|---|---|
| **Google Ads proxy policy** forbids hosted services that "solely re-expose" Ads API capabilities and expects per-customer Cloud projects | **High, existential for the Google Ads piece if misread** | Get written guidance from Google first. Design the Google Ads connector as value-added (joined to orders, guardrailed, not a raw GAQL pass-through) and support a **customer-owned Cloud project** mode from day one. See PRD-01 |
| Google Ask Advisor answers cross-Google questions for free | Medium | It will not see Meta, the ERP, or reconcile money. Position on "your money, all platforms", not "Google insights" |
| Meta extends its connector to include CAPI diagnostics and revenue | Medium | Meta will never reconcile against Google or the ERP |
| Cometly moves down-market for lead-gen | Medium | Compete on flat price, custom-stack ERP connectors, Grok support, assistant-first setup |
| Shopify brands are well served | Low (we do not target them first) | Shopify as a source later, not the wedge |
| LLM vendors build "analytics agents" natively | Medium, long-term | Our moat is the data contract and write-back, which the assistant vendors will not build per platform |

## 6. Options considered

| Option | For | Against | Verdict |
|---|---|---|---|
| A. Generic "connect GA4/GSC/Ads/Meta to any AI" connector | Fast to build, easy story | Commoditised, Google policy risk, race to the bottom on price | Reject as the product; keep as free tier |
| B. **Revenue truth layer for non-Shopify SMBs** (read + orders + reconciliation + guardrails, all assistants) | Matches our proven internal build, open gap, defensible | Needs per-ERP connectors; smaller TAM than Shopify | **Recommend as the wedge** |
| C. Signal-quality product (CAPI, GA4 MP, Data Manager API write-back) with AI as the interface | Directly lifts ad performance, so easiest ROI story | Crowded by Stape/Elevar/Cometly; harder setup | **Add as the paid upsell in phase 2** |
| D. Agency white-label | Higher ARPA, agencies buy tools | Agency tools are saturated (Whatagraph, AgencyAnalytics, TMR, Swydo) | Phase 3, via a "per business" price that agencies can stack |

## 7. A challenge to our own thesis

It is worth holding two uncomfortable possibilities in view:

- **The ROI story may be stronger in the write-back than in the chat.** Owners pay for "Meta now sees 30% more of your real purchases, so your CPA drops" more readily than for "you can ask Claude about ROAS". If early interviews say so, flip the order of PRD-04 and PRD-03.
- **Some of the market may prefer a done-for-you service.** Our teardown took two weeks for a technical founder. A non-technical SMB might want a $500 setup service plus a cheap subscription rather than a self-serve app. Test a concierge onboarding offer in the pilot before building heavy self-serve UI.

## 8. Kill or pivot criteria (first 90 days)

- Google will not confirm a compliant hosted path, and fewer than 30% of pilot users will create their own Cloud project with guided setup. → Drop hosted Google Ads, keep GA4/GSC, point Ads users to the official open-source server plus our order join.
- Fewer than 5 of 20 pilot businesses connect an order source within their first week. → The order join is not the wedge; re-test option C.
- Weekly active assistant queries per business under 3 after week 4. → Lead with the proactive brief and alerts, not chat.
