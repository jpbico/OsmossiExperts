# OsmossiExperts: Osmossi marketing agent team

This repo holds the agents and skills behind the **Osmossi Growth Engine**: a weekly
marketing cycle that brings traffic to the Osmossi store and turns it into sales.

Source map: whiteboard "Osmossi Growth Engine – Agents & Skills Map" (claude.ai artifact
`63ac1270-e2bc-4365-8f5a-bd843a7d2bc7`).

## The cycle

Learn the market → plan the offer → create content → attract buyers → convert on the site
→ measure & learn → keep customers. Learnings feed the next cycle.

| # | Stage | Agent (`.claude/agents/`) | Skills (`.claude/skills/`) |
|---|-------|---------------------------|----------------------------|
| 0 | Orchestration | `marketing-lead` | runs the weekly cycle, routes work |
| 1 | Understand the market | `market-intelligence` | competitor-scan, trend-scout, audience-insights, price-benchmark |
| 2 | Plan the offer | `growth-planner` | monthly-marketing-plan, hero-product-picker, gift-guide-bundle-builder, budget-split |
| 3 | Create content | `content-studio` | brand-voice-copy, photo-edit, reel-editor, pinterest-pin-builder, osmossi-brand-review |
| 4 | Attract buyers | `distribution-ads` | content-calendar-scheduler, meta-campaign-builder, creator-outreach, lead-capture |
| 5 | Convert on the site | `store-conversion` | product-page-optimizer, trust-offer-audit, checkout-recovery, dm-sales-assistant |
| 6 | Measure & learn | `insights` | weekly-kpi-report, funnel-diagnosis, ad-performance-review, test-log-next-bets |
| 7 | Keep customers | `loyalty` | post-purchase-thank-you, review-ugc-request, repeat-referral-offer |

## Shared context (every agent reads these first)

- `context/business.md`: what Osmossi sells, store, goals, partners
- `context/brand.md`: voice, language, visual identity, claims rules
- `context/audience.md`: who we sell to
- `context/approval-gates.md`: what always needs a human yes, and who approves it
- `context/content-formats.md`: format sizes, production steps, pre-post checklist
- `context/market-watchlist.md`: competitors, hashtags, moments calendar

Sections marked `TODO(input)` are still waiting on answers from the partners. Agents must not
invent facts to fill them; they say what is missing instead.

## House rules for all agents

1. **Human approval gates are hard stops.** Never publish a post, spend money, accept terms or
   change a price without an explicit yes from a partner. Prepare drafts; a partner approves.
2. **Osmossi ERP writes** always carry `requested_by` (Juan Pablo, Andrea or Lucelly). If it is
   unclear who asked, ask. Deletes need `confirm: true` and a partner's confirmation first.
   Finance in the ERP is read-only.
3. **Never guess Shopify IDs**: look them up first. Money is integer cents with a currency code.
   Times are New York.
4. Hand results to the next stage in the formats each agent's "Outputs" section names.
5. Cite where every market fact came from (URL, tool, date).
