---
name: competitor-scan
description: Scan Osmossi's competitors (Coper Shop, Lombia, Helecho Collective, Maracuyá) on their websites and Instagram for new products, prices, offers and messaging. Use for the weekly market brief or when a competitor makes a move.
---

# competitor-scan

Owner agent: `market-intelligence`. Runs weekly.

## Inputs
- Competitor list: `context/market-watchlist.md`
- Last week's brief in `docs/briefs/` (to report only what changed)

## Steps
1. For each competitor site, list its product pages (`firecrawl_map`), then scrape the home page,
   the bag collection and any sale or gift page (`firecrawl_scrape`, `maxAge: 0`).
2. Record for each: new products since last week, bag prices, active offers (discounts, free
   shipping, bundles, gift guides), headline messaging, and how they show origin and artisans.
3. Instagram: look at public profile and recent post pages through `firecrawl_scrape` or web
   search. Instagram often blocks automated reads; if a page fails, say so and use whatever the
   site, Pinterest or search results show instead. Never log in.
4. Compare with last week's brief and keep only changes and anything Osmossi should react to.

## Output format
A "Competitors" section in the weekly brief:

| Competitor | What changed | Price / offer | So what for Osmossi | Source (URL, date) |
|---|---|---|---|---|

At most 5 rows. If nothing changed, one line saying so.

## Rules
- Read-only: no logins, follows, likes, messages or comments.
- Report prices as listed, in USD, with the date seen.
