---
name: hero-product-picker
description: Choose Osmossi's hero products for a week or month. Opalo beaded bum bags come first, then the other bags; only in-stock pieces; matched to the moment and to current trends. Use when setting the week focus or the monthly plan.
---

# hero-product-picker

Owner agent: `growth-planner`.

## Priority (set by Juan Pablo, 2026-11-19)
1. **Opalo beaded bum bags** ($499): the hero line.
2. **All other bags**: Opalo mini bags, then Pausayu bags (Macuira, Pompoms, Camila, Kai, Half
   Moon, Full Moon, Fresh Romance).
3. Hats and home decor support gift guides and moments; they are not heroes.

## Steps
1. Pull bags live from Shopify: `graphql_query` (e.g. `products(first: 50, query: "product_type:'Handwoven Bags' AND status:active")` with `title handle totalInventory priceRangeV2`).
   Keep only `totalInventory ≥ 1`.
2. For each candidate, score:
   - **Line priority** (above): bum bag 3, other bag 2, other 1.
   - **Moment fit**: colors and occasion for the month (red and gold for the holidays, bright
     colors for summer, etc.). 0–2.
   - **Trend fit**: matches a strong ERP trend (for example, "bold color, sparkle" → beaded
     pieces). 0–2.
   - **Freshness**: not featured as hero in the last 4 weeks (`list_posts`). 0–1.
3. Pick **2 heroes per week** (at least 1 bum bag) and up to 2 supporting bags.
4. If a hero sells out, swap in the next-highest score the same day and tell `content-studio`.

## Output format

| Rank | Product (link) | Line | Stock | Why now |
|---|---|---|---|---|

Links go to the product page: `https://www.osmossi.com/product/<handle>` (storefront route).
