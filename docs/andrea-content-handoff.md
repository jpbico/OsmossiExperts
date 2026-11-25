# How Andrea picks up a Content Studio draft

The ERP post is the single place a draft lives. The Content Studio agent writes the draft into the
post; Andrea opens it from her usual claude.ai project ("AL Media Editor for Osmossi"), refines it
with Claude there, and approves it by setting the post to **ready**.

```
Content Studio agent             ERP post (status: idea)              Andrea's claude.ai project
builds design + copy  ───────►   caption = option A          ───────►  "Open the draft for Nov 28"
brand review PASS                notes = brief + DRAFT v1              edits in chat + Canva
                                 Canva link(s)                         final export → SharePoint
                                                              ◄───────  caption updated, status → ready
```

## One-time setup (Andrea)
1. In claude.ai, turn on these connectors for her account: **Osmossi ERP**, **Canva**,
   **Microsoft 365** (SharePoint).
2. In the project "AL Media Editor for Osmossi", add the block below to the end of the project
   instructions.

## Paste into the project instructions

```
# CONTENT STUDIO DRAFTS (from the Osmossi agents)
- Drafts for upcoming posts live in the Osmossi ERP calendar. When I say "open the draft for
  <date or title>", find that post with list_posts / get_post and read its caption and the
  "CONTENT STUDIO DRAFT" block in its notes (Canva links, assets used, option B, hashtags,
  alt text, Pinterest copy, brand review, open questions).
- Open the Canva design from the link and work on that design. Do not start a new one unless
  I ask.
- When I make changes, show me the new caption or design first. Only when I say "final" or
  "approved":
  1. update the ERP post: caption = the final caption; add a line "--- FINAL <date> by Andrea ---"
     with the final Canva link to the end of the notes, keeping everything above it;
  2. export the final files with the naming osmossi_[product]_[platform]_[ratio]_[v#] and save
     them to SharePoint Leales › 4 - Marketing › Biblioteca generar contenido › Final Posts ›
     <YYYY> › <MM> (create the folder if it is missing). Files over 1 MB: give me the Canva
     download link and remind me to save it there by hand;
  3. set the post status to "ready".
  Always use requested_by: "Andrea".
- Never set a post to "posted" yourself; I do that after publishing.
```

## Day to day
1. On Monday the agent tells Andrea which drafts are waiting (date, title, Canva link).
2. Andrea opens a chat in the project: *"Open the draft for the Nov 28 carousel."*
3. She asks for changes in plain words, or edits the design in Canva directly.
4. She says *"final"*: Claude saves the caption, files the export in `Final Posts\2026\11`
   and sets the post to **ready**.
5. After publishing on Instagram or Pinterest, she sets the post to **posted**.
