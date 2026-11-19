---
name: monthly-marketing-plan
description: Write Osmossi's monthly marketing plan in the November 2026 ERP format (topic, subjects, weekly funnel, post mix, conversion rules, KPI targets, decisions pending), built on the moments of that month (US and Latin American holidays) and the previous month's results, then save it to the ERP month plan. Use by the 20th for the next month.
---

# monthly-marketing-plan

Owner agent: `growth-planner`.

## Inputs
- Last month's plan: `get_month_plan` (format reference: `2026-11`)
- Last month's results: latest `docs/reports/kpi-*.md`, `get_store_results`, `get_category_mix`,
  `list_posts`; `docs/test-log.md`
- Moments in the month: `context/market-watchlist.md` (US, CO, VE, MX)
- Trends and opportunities: latest `docs/briefs/market-*.md`, `list_trends`
- Hero products: `hero-product-picker`; budget: `budget-split`

## Steps
1. **Review last month:** which targets were hit or missed, and the one or two reasons why.
   Carry forward what worked; drop what didn't.
2. **Pick the month's moments:** every holiday in the month for US, Colombia, Venezuela and
   Mexico, plus gifting deadlines (carrier holiday cutoffs in December).
3. **Write the theme:** a short name + one sentence on the business goal for the month.
4. **Three subjects:** trust (craft, time, tradition) · desire (occasions, styling) · reason to
   buy now (the moment). Rewrite them for the month.
5. **Weekly funnel:** W1 Discover→Understand · W2 Understand→Connect · W3 Connect→Consider ·
   W4 Purchase. Shift the purchase week to land 7–10 days before the main moment.
6. **Post mix and formats:** apply "Choosing the mix" below.
7. **Conversion rules:** carry the standing rules (`context/brand.md` wording rules) and add
   any for this month (offer, order-by date).
8. **KPI targets:** set each target from last month's actuals (not from hope). Where there is
   no baseline yet, say "first baseline".
9. **Decisions pending:** list what a partner must decide, each with a recommendation
   (see `docs/plans/*-decisions.md`).
10. Write the draft to `docs/plans/YYYY-MM-plan.md` and send it to Juan Pablo. After his yes,
    save it with `set_month_plan` (`topic` = theme + sentence + cadence; `subjects` = the rest,
    same layout as November).

## Choosing the mix
There is no fixed mix: choose the one most likely to bring new people and store visits, then let
results move it.

**Starting point** (until we have 4 weeks of data):

| Format | Per week | Job |
|--------|----------|-----|
| Reels | 2 | Reach new people (the main growth lever for a new account) |
| Carousels | 1–2 | Saves and shares; product detail and artisan story |
| Single image | 0–1 | Hero product moments only |
| Stories | daily | Link taps to product pages, polls, new arrivals |
| Pinterest pins | 8–10 | Long-term traffic: 2–3 pins per hero product |

Pillars: rotate all six from `context/brand.md`; at least 2 posts a week show **styling** (how
she wears it), because "not stylish" is a top objection; at least 1 shows the **artisan or craft
detail**, because "it's common" is the other.

**Each month, move the mix by evidence:** from the `insights` report, add one slot to the format
and pillar with the best link-tap rate (taps ÷ reach) and remove one from the worst. Change only
one or two things a month so the effect stays readable. Record the change in `docs/test-log.md`.

## Output format
Same structure as the November 2026 ERP plan:
- **topic:** "<Theme> — <one-sentence goal>. <N> feed posts (<days>), daily Stories, building toward <moments with dates>."
- **subjects:** three subjects · Funnel · Mix (with post numbers) and formats · Conversion rules · KPI targets · Weekly review · Decisions pending.
