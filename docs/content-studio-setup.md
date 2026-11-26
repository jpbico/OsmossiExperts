# Content Studio: Canva + SharePoint setup checklist

Checked 2026-11-26. Items marked **[you]** need a partner; **[done]** are configured.

## How files can move today
| Route | Works? | Note |
|-------|--------|------|
| Shopify product photos → Canva | Yes | Public CDN link + `upload-asset-from-url` (used in the trial) |
| SharePoint photos → this session | Yes | The connector saves JPG/PNG at full size (1080×1080 checked) |
| SharePoint photos → Canva | **Yes** | Tested 2026-11-26: gift box photo uploaded at full 1080×1080 |
| SharePoint videos → anywhere | No | The connector can't read video files |
| Canva export → this session | **Yes** | Tested 2026-11-26: JPG export downloaded (230 KB) |
| This session → SharePoint | Yes, up to 1 MB | JPG posts fit; Reels (MP4) don't |

## Checklist
1. **[done]** Network: `www.canva.com` and `export-download.canva.com` allowed in the "Osmossi ERP" environment.
2. **[done] Allow the transfer commands:** rules added to `.claude/settings.json` by Juan Pablo, 2026-11-26. Approve a permission rule so the agent can run `curl` to those two hosts
   (upload SharePoint photos to Canva; download finished exports for SharePoint).
3. **[in review] Create Canva brand templates** (5 drafts in Canva › Productos Pagina Web › Templates; see `docs/canva-brand-templates.md`). (Canva Pro: owner/admin/Brand Designer can publish).** One per format,
   with data fields so the agent fills them instead of generating layouts from scratch:
   - Reel/Story 9:16 scene (fields: `photo`, `headline`), Reel end card (`offer`, `dates`)
   - Feed 4:5 single and carousel slide (`photo`, `headline`, `line2`)
   - Pinterest 1000×1500 (`photo`, `overlay`)
   Logo, fonts and colors locked in each template. The agent then uses `autofill-design`.
4. **[done] Canva folders:** by brand, `Productos Pagina Web › Opalo|Pausayu › Borradores|Finales › 2026 ›
   11 Noviembre / 12 Diciembre` (IDs in the agent file).
5. **[you] Video clips.** No connector can move video into Canva for the agent (SharePoint can't read video;
   Canva's Google Drive / Dropbox / OneDrive connections don't accept video, per Canva Help 2026-11-26).
   Upload clips from the phone in the Canva app (Uploads, or Add media in the video editor) so they're ready
   when Andrea finishes a Reel. Optional: clips uploaded to Shopify › Content › Files get a public link the
   agent can import into Canva itself.
6. **[you] Raw library hygiene.** Name raw files with the product and scene ("Ravello - worn - street.jpg"),
   and export iPhone photos as JPG (HEIC often fails in Canva).
7. **[done 2026-11-26] Andrea's project.** Turn on the Osmossi ERP, Canva and Microsoft 365 connectors in her claude.ai
   account and paste the block from `docs/andrea-content-handoff.md`.
8. **[you] Artisan facts** (names, communities, stories, photo consent) for SharePoint.
9. **[you] Fix live material:** the Black Days Ravello slide and the Shopify collection image say "One of a kind".
