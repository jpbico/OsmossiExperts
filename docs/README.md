# OsmossiExperts — Research Docs

Research for a mobile-friendly, installable web app (PWA) on Netlify. It has an **Instagram post module** and an **Agent Console**. These docs are the shared context for the team of expert agents that will build it. Each file covers one topic, says where its facts come from, and ends with open questions.

Research date: 2026-11-26. Re-check numbers that may change (limits, model IDs, API versions) before you build on them.

## Index

| # | File | Topic |
|---|------|-------|
| 1 | [01-use-case-instagram-module.md](01-use-case-instagram-module.md) | Use case, the Instagram Posts Generator API, publishing through the Instagram Graph API, and a scoring model from the API fields |
| 2 | [02-pwa-installability.md](02-pwa-installability.md) | What makes a PWA installable on iPhone and Android; when to use a native wrapper (Capacitor) |
| 3 | [03-architecture-netlify.md](03-architecture-netlify.md) | Static Netlify site + Netlify Functions + Netlify Blobs: layout, limits, env vars |
| 4 | [04-agent-console.md](04-agent-console.md) | Agent Console: agent list, chat, propose → approve → commit → redeploy; security; long jobs |
| 5 | [05-testing-accessibility.md](05-testing-accessibility.md) | Test strategy and accessibility (WCAG 2.2 AA) |

## Key findings (read these first)

1. **The "MCP connector" is a plain REST API, and it does not post to Instagram.** `instagram-posts-generator.vercel.app` renders 1080×1080 post images from 4 templates (9GAG, Pubity, Tatva India, Terribly Dank Tales) through `POST https://instagram-posts-generator.vercel.app/api/v1/images`. It needs no key and allows 100 requests per minute per IP. Its docs describe no MCP server. Posting to Instagram needs a separate integration: the **Instagram Graph API (Content Publishing)**, which requires OAuth, a Business or Creator account, a **public JPEG URL**, and a two-step container → publish call (max 100 API posts per 24 h). See doc 1.
2. **Item (1) of the brief, "score a beach day", does not fit this API.** No field in it describes weather or beaches; that line looks copied from another project's template. Doc 1 offers a **post-readiness score** built from the fields the API really has. Confirm which one you want.
3. **Netlify limits shape the Agent Console.** A normal or streaming function stops at **60 s** (a streamed response can be up to 20 MB). A **background function** can run for **15 min**, but it answers the caller with 202 at once and its request payload is capped at 256 KB. Long agent jobs must therefore run as background jobs, with their state in Netlify Blobs. See docs 3 and 4.
4. **A commit to `main` starts a production deploy.** The Approve button should write one atomic commit through the Git Data API, using a fine-grained token limited to this repository (Contents: read/write). The server must check every path against an allow-list. See doc 4.
5. **On iOS, users install from the Share menu.** iOS has no `beforeinstallprompt` event. Web Push works only after the app is added to the Home Screen (iOS 16.4+). A native wrapper is needed only for app-store distribution or native-only APIs. See doc 2.

## Decisions needed from the owner

- Scoring: post-readiness score (proposed) or something else?
- Which Instagram account will post (it must be a Business or Creator account), and who creates the Meta app?
- Should approved agent changes go straight to `main` (production deploy), or to a branch with a Deploy Preview first? Doc 4 recommends a branch first.
- Default model per agent (`opus` → `claude-opus-5-5` unless an agent's file says otherwise).
