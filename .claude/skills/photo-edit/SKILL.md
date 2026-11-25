---
name: photo-edit
description: Choose and grade Osmossi product photos to the house standard (natural, warm, true-to-life) and build feed photos, carousels, Stories and Pins in Canva from the Osmossi brand kit, using raw files from the SharePoint content library. Use when preparing any image post.
---

# photo-edit

Owner agent: `content-studio`.

## Inputs
- Raw files: SharePoint `4 - Marketing\Biblioteca generar contenido` (search by product name;
  files are named by product, e.g. "Raining Amber.jpg", "Ravelo Box Nature.heic").
- Brand kit "Osmossi" in Canva; standards in `context/brand.md`; sizes in `context/content-formats.md`.

## Steps
1. **Find candidates**: search the library for the product and the brief's scene. List them with
   a quick assessment: what works, what to fix, and the strongest shots. If nothing usable exists
   (blurry, color off, missing scene), say so and write a short shot list for a reshoot.
2. **Bring them into Canva**: `upload-asset-from-url` (or `create-upload-url`) for each chosen
   file. HEIC files may not import; if one fails, ask for a JPG export.
3. **Build the design from the brand kit**: Beige Sand and Green as main colors; Playfair Display
   for headlines, Lora for body; logo exactly as supplied, small, with enough contrast.
4. **Grade** (in Canva's photo adjustments): slightly warm white balance, gentle contrast, lifted
   shadows, controlled highlights, moderate saturation. The product color must match the real
   piece. Where Canva can't do an adjustment, write exact settings for Andrea (e.g. Temp +8,
   Highlights −30, Shadows +20).
5. **Text**: minimal, inside safe zones (Stories: not in the top ~250 px or bottom ~350 px).
6. **Sizes**: feed and carousel 1080×1350; Story 1080×1920; Pin 1000×1500. Use `resize-design`
   for other formats of the same post.
7. **Name** every export `osmossi_[product]_[platform]_[ratio]_[v#]`.

## Carousel order (default)
Hook → detail close-up → worn or used → origin → logo + "Shop at osmossi.com" on Green.

## Never
Alter the product's shape, pattern or color; recolor or stretch the logo; use other fonts.
