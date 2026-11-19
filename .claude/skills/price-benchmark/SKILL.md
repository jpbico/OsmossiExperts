---
name: price-benchmark
description: Compare Osmossi bag prices with comparable bags at Coper Shop, Lombia, Helecho Collective and Maracuyá, and check Osmossi sits between mid and premium. Use monthly, before pricing proposals, bundles or promotions.
---

# price-benchmark

Owner agent: `market-intelligence`. Runs monthly or on request.

## Inputs
- Osmossi bags with live prices: Shopify `search_products` with `product_type:"Handwoven Bags"`
- Competitor sites: `context/market-watchlist.md`

## Target position
Osmossi aims **between mid and premium**: above the competitors' median for a comparable bag
and at or below their top price.

## Steps
1. Group Osmossi bags into comparable types:
   - Wayuu mochila (Pompoms, Camila)
   - Palm tote (Macuira line)
   - Clutch / evening (Half Moon, Fresh Romance)
   - Statement bag (Full Moon, Kai)
   - Beaded bum bag (Opalo)
   - Beaded mini bag (Opalo)
2. For each type, collect competitor bags of the same type with price, size and material.
   Use the same day for all prices; note sale prices separately from full prices.
3. Per type, compute competitor min, median and max, and Osmossi's position.
4. Flag each Osmossi type as **below target** (under the median), **on target** (median to
   max) or **above market** (over the max), with what justifies or doesn't justify the gap
   (handwork hours, beadwork, uniqueness).

## Output format

| Type | Osmossi price | Competitor min / median / max | Position | Note |
|---|---|---|---|---|

Followed by sources (URL and date for each competitor price).

## Rules
- This skill **proposes**; it never changes a price. Price changes are an approval gate
  (Juan Pablo).
- If fewer than 3 comparable competitor bags exist for a type, say so instead of computing a median.
