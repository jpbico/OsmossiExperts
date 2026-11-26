# Content Studio: Canva + SharePoint setup checklist

Checked 2026-11-26. Items marked **[you]** need a partner; **[done]** are configured.

## How files can move today
| Route | Works? | Note |
|-------|--------|------|
| Shopify product photos → Canva | Yes | Public CDN link + `upload-asset-from-url` (used in the trial) |
| SharePoint photos → this session | Yes | The connector saves JPG/PNG at full size (1080×1080 checked) |
| SharePoint photos → Canva | Pending approval | Needs a direct upload to Canva from this session (hosts allowed; command needs a permission rule) |
| SharePoint videos → anywhere | No | The connector can't read video files |
| Canva export → this session | Pending approval | Same permission rule |
| This session → SharePoint | Yes, up to 1 MB | JPG posts fit; Reels (MP4) don't |

## Checklist
1. **[done]** Network: `www.canva.com` and `export-download.canva.com` allowed in the "Osmossi ERP" environment.
2. **[you] Allow the transfer commands.** Approve a permission rule so the agent can run `curl` to those two hosts
   (upload SharePoint photos to Canva; download finished exports for SharePoint).
3. **[you] Create Canva brand templates (Canva Pro: owner/admin/Brand Designer can publish).** One per format,
   with data fields so the agent fills them instead of generating layouts from scratch:
   - Reel/Story 9:16 scene (fields: `photo`, `headline`), Reel end card (`offer`, `dates`)
   - Feed 4:5 single and carousel slide (`photo`, `headline`, `line2`)
   - Pinterest 1000×1500 (`photo`, `overlay`)
   Logo, fonts and colors locked in each template. The agent then uses `autofill-design`.
4. **[you] Canva folders for the agent.** Canva has two "Borradores" and two "Finales" folders. Pick one
   drafts folder and one finals folder (or confirm `Instagram › 2026 › <Mes>`), so drafts stop landing in the root.
5. **[you] Video clips.** SharePoint videos can't reach Canva through the connector. Options: upload new clips
   straight to a Canva folder (e.g. `Clips`) from your phone, or keep adding them by hand while finishing a Reel.
6. **[you] Raw library hygiene.** Name raw files with the product and scene ("Ravello - worn - street.jpg"),
   and export iPhone photos as JPG (HEIC often fails in Canva).
7. **[you] Andrea's project.** Turn on the Osmossi ERP, Canva and Microsoft 365 connectors in her claude.ai
   account and paste the block from `docs/andrea-content-handoff.md`.
8. **[you] Artisan facts** (names, communities, stories, photo consent) for SharePoint.
9. **[you] Fix live material:** the Black Days Ravello slide and the Shopify collection image say "One of a kind".
