# Option review: add static ad creative, and niche down to one lead-gen vertical

Date: 3 Nov 2026. Builds on `03-gap-and-opportunity.md` and `04-positioning.md`. Prices are mostly third-party snapshots because few creative vendors publish them; sources inline.

## 1. Short answer

- **Niching to lead-gen in one vertical: yes.** It sharpens every part of the product (metric definitions, CRM connectors, compliance rules, templates, word of mouth) and moves us away from the crowded Shopify and generic-connector markets.
- **Static ad generation as a feature: only in a specific form.** Generating images and copy is already free inside Meta and Google and cheap in Canva. What nobody does is **score creatives by what happened after the lead**: which ad produced site visits, quotes and deposits. Build that, and use generation as the last step, not the product.
- **Recommended vertical: renovation and interior design firms in Singapore**, with Sofnade (events and catering) as the dogfood account, because both run the same sales motion: enquiry, then quote, then deposit.

## 2. Static creative: what is already free or cheap

| Source | What it does | Price | Uses revenue or CRM outcomes? |
|---|---|---|---|
| Meta Advantage+ creative and gen-AI tools | Text variations, background generation, image expansion, animation. Nearly 10M advertisers used at least one in Q3 2026 ([medianews4u](https://www.medianews4u.com/metas-q3-ad-revenue-rises-26-to-63-1-billion-as-ai-boosts-campaign-performance/)) | Free | Pixel/CAPI events only |
| Google Asset Studio (Gemini) | Image, text, video; 1-click creative A/B tests since May 2026 ([ppc.land](https://ppc.land/asset-studio-gets-gemini-omni-and-1-click-creative-testing-at-gml-2026/)) | Free in Google Ads | Google conversions only |
| Google Pomelli | Website URL to campaign creative ([buildfastwithai](https://www.buildfastwithai.com/blogs/what-is-google-pomelli-ai)). Singapore availability unverified | Free | No |
| Canva Grow 2.0 | Create, launch and optimise ads on Meta, TikTok, LinkedIn ([BusinessWire](https://secure.businesswire.com/news/home/20260625870253/en/Introducing-Canva-Grow-2.0-Create-Launch-and-Optimize-Ads-in-One-Place)) | Pro ~$18/mo | Platform metrics, no Google, no CRM |
| AdCreative.ai | Generation + predictive scoring | $39 to $999/mo, credits | Platform metrics |
| Omneky, Atria, Creatify, Pencil, Predis | Generation, some with ad-account insights | $14 to $959/mo | Platform metrics |
| Motion | Creative analytics; GA/Northbeam attribution on Pro | $999+/mo | Third-party attribution, not CRM |
| Madgicx | Meta optimisation + server-side tracking | Spend-tiered; tracking $49/mo | Server-side Meta only |

Evidence on performance is thin and mixed. One randomised field study found the best AI banner beat a human stock-photo ad by up to 50% CTR ([Hartmann et al., IJRM 2025](https://ideas.repec.org/a/eee/ijrema/v42y2025i1p13-31.html)); Taboola and Columbia found parity on average, with AI ads winning only when they did not look AI-made ([IT Brief](https://itbrief.com.au/story/study-finds-ai-ads-rival-humans-when-visuals-build-trust)). **No study we found measures cost per qualified lead or revenue.** That missing measurement is the opening.

### What to build instead of a generator: the revenue-scored creative loop

1. **Score**: join each ad and creative id to downstream outcomes from the CRM/WhatsApp pipeline (lead → consult → quote → deposit), and rank creatives by cost per *quote* and per *deposit*, not by CTR or platform CPL.
2. **Explain**: describe what the winners share (angle, room type, price anchor, before/after, offer, format), in the assistant and in the weekly brief.
3. **Brief and generate**: produce new static variants and copy from the winners, inside the vertical's compliance rules, then hand them to Meta/Google's native tools or Canva (via their MCPs) or upload directly as paused ads for approval (PRD-05 guardrails).
4. **Learn**: tag every variant so the next cycle scores it against real outcomes.

Generation is a commodity input; the score is ours.

## 3. Choosing the vertical

Benchmarks are US 2026 LocaliQ/WordStream ([search](https://localiq.com/blog/search-advertising-benchmarks/), [Facebook](https://localiq.com/blog/facebook-advertising-benchmarks/)) unless marked.

| Vertical | Google CPL | Meta CPL | Ticket size | Ad restrictions | Incumbents | Verdict |
|---|---|---|---|---|---|---|
| **Renovation / interior design** | $90.92 | $42.95 | SG 4-room S$51K to S$80K ([Qanvast via Lemon8](https://www.lemon8-app.com/@qanvast/7466035679378620935)) | None major | Qanvast (leads marketplace); trades served by ServiceTitan, but design-build and ID firms are not | **Recommend (commercial beachhead)** |
| Events / catering / weddings | ~$30.57 (restaurants proxy) | n/a | S$2K to S$30K per event | None | Fragmented; agencies | **Dogfood (Sofnade)**, small budgets limit revenue |
| Tuition / enrichment | $77.48 | $26.31 | Multi-year enrolment | Parents not minors; MOE rules unchecked | Unclear SG systems | Strong runner-up |
| Dental / medical aesthetics | $72.97 / $40.04 | $61.56 / $32.14 | High | **Meta health restrictions limit lower-funnel optimisation; MOH rules** | Weave, Zenoti, Mindbody | Avoid first: restrictions blunt write-back, our strongest lever |
| Legal | $131.63 | n/a | High | Bar rules | Scorpion, Lawmatics, Clio | Avoid: agency-saturated |
| Real estate agents | $102.51 | $13.74 | High per deal | Special Ad Categories (US/CA/EU); CEA rules in SG | Portals | Avoid: individual agents, small budgets |
| Fitness | $67.36 | $27.11 | Low | None | Mindbody | Avoid: low ticket, lock-in |

### Why renovation and interior design
- One signed project (S$50K+) pays for years of subscription, so price can sit in the S$199 to S$499/mo range rather than $29.
- Leads are expensive and quality varies wildly, so "which ads bring *quotes*, not just leads" is a painful, money-shaped question.
- No Special Ad Category or health restrictions, so CAPI and Google offline write-back work fully.
- Highly visual: before/after, room types and styles make the creative loop concrete.
- Singapore has thousands of registered renovation contractors (6,697 ACRA-registered in 2022 per an [MTI written reply](https://www.mti.gov.sg/newsroom/written-reply-to-pq-on-renovation-contractors), attribution unverified), the same playbook extends to AU, UK and US design-build firms.

### Why Sofnade stays in the plan
Catering is not the best market to sell into (budgets are smaller), but it shares the exact mechanics: enquiry, quote, deposit, WhatsApp-heavy, long gap between deposit and the event. It lets us build and prove the pipeline end to end on an account we control, with Linhan's weekly Sofnade reporting as the existing baseline.

## 4. What changes in the product

| Area | Change |
|---|---|
| Revenue link (PRD-03) | Order sources become **pipeline sources**: stages lead → consult/site visit → quote → signed → deposit → progress payments. Add **WhatsApp Business** (click-to-WhatsApp ads, lead id via prefilled message code), HubSpot, Zoho, Google Sheets, Qanvast LMS exports |
| Signal loop (PRD-04) | Write back **milestone events** to fit platform windows: consult within Meta CAPI's 7 days; quote/contract within Google's 90 day click window (63 days for enhanced conversions for leads), via the Data Manager API. Investigate Meta's conversion events for business messaging for WhatsApp leads **[verify]** |
| Truth layer (PRD-02) | Vertical metric pack: cost per qualified lead, cost per site visit, cost per quote, quote-to-sign rate, cost per signed project, pipeline value by campaign. Lag-aware: "still converting" until the median quote-to-sign time has passed |
| New PRD-07 | Revenue-scored creative loop (§2) with renovation templates (room types, HDB/condo/landed, styles) and compliance checks |
| Brief (PRD-06) | Delivered on **WhatsApp first**, then email. Renovation owners live in WhatsApp more than in Claude |
| Positioning | From "revenue truth layer for your AI assistant" to **"Know which ads bring signed projects, and make more of them"**. The assistant becomes a delivery channel, not the headline |
| Pricing | Per firm: Core S$199/mo (reporting, pipeline, brief), Growth S$399/mo (+ write-back, creative scoring), Studio S$699/mo (+ creative generation with approval, multiple brands). Compare: agency retainers S$1.5K+/mo, Qanvast prepaid lead credits, HighLevel $97 to $497 |

## 5. Challenges to this direction

- **We would be trading a horizontal tool for a narrow one.** If Singapore renovation turns out to be a few hundred firms that spend seriously, the ceiling is low until we export the playbook. Mitigation: design the vertical pack as configuration, so "tuition" or "AU design-build" is a new pack, not a new product.
- **The AI-assistant angle weakens.** Most renovation owners will not add an MCP connector to Claude. The product becomes closer to a vertical marketing OS delivered on WhatsApp, with assistant access for the marketer or agency. That may be a better business, but it is a different one from the original brief.
- **Agencies are the real buyers.** In Singapore, most renovation firms outsource ads. Selling to the 20 agencies that serve them may beat selling to 2,000 firms. Test both in discovery.
- **WhatsApp breaks attribution.** Many leads arrive as chats with no form id. Prefilled message codes and click-to-WhatsApp tracking solve some of this, not all. This is the riskiest technical assumption; prototype it on Sofnade first.
- **Creative in renovation relies on real project photos.** Generated interiors that look fake hurt trust (consistent with the Taboola finding). The creative loop should prioritise the firm's own portfolio photos, with AI for layout, copy, crops and variants, not fabricated rooms.

## 6. Proposed next 30 days

1. Ten discovery calls: 6 renovation/ID firms, 4 agencies that run their ads. Questions: how leads arrive, what CRM, what they pay agencies, would they pay for "cost per signed project".
2. Prototype the WhatsApp lead-id capture and milestone write-back on Sofnade.
3. Pull 90 days of Sofnade ads and enquiries and produce a manual "revenue-scored creative" report as the demo artefact.
4. Decide go / no-go on the vertical using the kill criteria in `03-gap-and-opportunity.md`, plus: at least 3 of 6 firms say they would pay S$199+/mo.
