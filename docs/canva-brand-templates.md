# Osmossi Canva brand templates

Drafts built 2026-11-26 from brand kit "Osmossi" (`kAHOo0oQgAk`), filed in Canva
`Productos Pagina Web › Templates`. Status: **draft, waiting for Juan Pablo / Andrea review**.

## Drafts
| Template | Canva design | Review notes (brand check) |
|----------|--------------|----------------------------|
| Feed post | [DAHb_JjWW5U](https://www.canva.com/d/ZbVptDFyb0BYVQm) | PASS |
| Carousel (5 pages) | [DAHb_Nzf7xM](https://www.canva.com/d/fa74zQFq5HApQF5) | FIX: page 1 cream text is low-contrast on the light door (add a soft shade or darker text). Check pages 2–5. Page 4 added two lines of its own ("Every stitch carries a story. Rooted in Colombia, made to travel.") |
| Story (3 pages) | [DAHb_EhJqKA](https://www.canva.com/d/IJegKs6JTPX8MS1) | FIX: page 1 logo sits in the top 250 px (covered by Instagram's bar): move it down or remove it (logo belongs on page 3). Check page 3 logo renders as the supplied wordmark |
| Reel storyboard (4 pages) | [DAHb_NfiUqQ](https://www.canva.com/d/xx6_MApfFX4X8u1) | FIX: cover hook text overlaps the bag: move it above or below the product. Check pages 2–4 |
| Pinterest Pin | [DAHb_Atrs-o](https://www.canva.com/d/M7lUgwWOhJGu3fA) | Sample overlay says "Handwoven" on a beaded bag (it's a field, filled per product) |

## Steps to go live
1. Review each draft in Canva (layout, logo, fonts, colors); edit by hand if needed.
2. Approve → the agent publishes each as a brand template (`publish-brand-template`).
3. The agent tags the elements below with autofill field labels and republishes.
4. The Content Studio agent then fills them with `autofill-design` instead of generating
   layouts from scratch.

## Templates and autofill fields
| Template | Size | Pages | Autofill fields (label → type) |
|----------|------|-------|--------------------------------|
| Feed post | 1080×1350 | 1 | `photo` image · `headline` text · `subline` text |
| Carousel | 1080×1350 | 5 | `hook_photo` image · `hook` text · `detail_photo` image · `detail_caption` text · `worn_photo` image · `worn_caption` text · `origin_photo` image · `origin_title` text · `origin_lines` text · `cta` text |
| Story sequence | 1080×1920 | 3 | `hook_photo` image · `hook` text · `product_photo` image · `product_name` text · `product_line` text · `moment_photo` image · `moment_title` text · `moment_line` text |
| Reel storyboard | 1080×1920 | 4 | `cover_photo` image · `hook` text · `scene2_photo` image · `scene2_caption` text · `scene3_photo` image · `scene3_caption` text · `end_title` text · `end_line` text |
| Pinterest Pin | 1000×1500 | 1 | `photo` image · `overlay` text · `subline` text |

The logo, fonts, colors and layout stay locked (not fields).
Reel clips are added by Andrea after autofill (Canva can't place video from the agent's sources).
