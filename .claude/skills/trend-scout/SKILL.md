---
name: trend-scout
description: Find trends for handmade Colombian bags and decor on Pinterest, Google, Etsy, Amazon and Instagram, plus upcoming US, Colombian, Venezuelan and Mexican holidays, and push the worthwhile ones into the Osmossi ERP Trends board. Use weekly and before monthly planning.
---

# trend-scout

Owner agent: `market-intelligence`. Runs weekly and writes to the ERP Trends board.

## Inputs
- Hashtags, accounts and the moments calendar: `context/market-watchlist.md`
- Existing trends: `mcp__Osmossi_ERP__list_trends`
- The catalog (to say which product fits): Shopify `search_products`

## Steps
1. **Moments:** list moments in the next 6 weeks from the calendar. Each one becomes a trend
   note with the products that fit and when to post.
2. **Search signals:** web search for current data on Pinterest Trends / Pinterest Predicts,
   Google Trends, Etsy trend reports and Amazon gift lists, for terms such as "wayuu bag",
   "beaded bag", "woven bag", "handmade gift", and "natural fiber decor".
3. **Instagram:** check the watchlist hashtags and accounts for what is getting attention
   (formats, colors, styling, sounds). If Instagram blocks the read, say so.
4. **Judge each finding:** `strong` if the source has a number and is recent (≤ 90 days) and
   US-relevant; `weak` otherwise (old, non-US, third-party, or no number).
5. **Deduplicate:** if a trend already in the ERP covers it, `update_trend` (new stat, new
   `checked_on`) instead of adding a new one.
6. **Write** each new trend with `create_trend`:
   - `source`: `google`, `pinterest`, `etsy`, `amazon`, `instagram` or `holiday` (holiday and
     cultural moments). Use the one the evidence came from. If the ERP refuses `instagram` or
     `holiday`, the ERP app has not been republished yet: put those in the brief and say so.
   - `headline`, `headline_stat`, `meaning` (one or two plain sentences on what it means for
     Osmossi), `try_this` (one concrete action naming a product), `source_url`, `source_name`,
     `checked_on` (today), `strength`.
   - `requested_by`: the partner who started this run; for the scheduled weekly run, `Juan Pablo`.

## Output format
- ERP trend notes as above (match the tone of the existing notes: short, plain, one action).
- A "Trends" section in the weekly brief listing what was added or updated.

## Rules
- Every trend has a source URL. No source, no trend.
- At most 5 new trends per week: keep the ones Osmossi can act on.
