# 1. Use case: the Instagram post module

## 1.1 What the user wants

On a phone, from the installed app:

1. Pick a template and fill in title, background image, colours and so on.
2. Preview the 1080×1080 image.
3. Write a caption.
4. Tap **Publish**. The post appears on the brand's Instagram account (now, or at a scheduled time).

Two separate services are needed, because the generator only makes images:

| Step | Service | Auth |
|------|---------|------|
| Render the image | Instagram Posts Generator API | None |
| Publish to Instagram | Instagram Graph API, Content Publishing | OAuth token for a Business or Creator account |

## 1.2 The Instagram Posts Generator API

Sources: <https://instagram-posts-generator.vercel.app/> (app), <https://instagram-posts-generator.vercel.app/docs> (API docs), the directory entry at <https://apidirectory.dev/apis/instagram-posts-generator>, and the directory the user found it in, <https://mcp-api.netlify.app/>.

**What it is.** A Next.js app on Vercel that turns templates modelled on popular Instagram pages into post images. Its docs describe a **REST endpoint**, not an MCP server. The directory lists it as *auth: none, free, HTTPS, CORS: yes*. We found no MCP endpoint in its docs or page code. (Our cloud container could not reach the host directly, so we did not test `/mcp`-style paths. Treat "MCP connector" as **not confirmed**.)

**Endpoint** (from `/docs`):

```
POST https://instagram-posts-generator.vercel.app/api/v1/images
Content-Type: application/json
```

- Payload: `name` (template name, string), `canvas` (`{ width, height }`), plus `params` (template-level fields, which "may vary" by template).
- Rate limit: **100 requests per minute per IP**. Response headers `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`.
- Errors: `200` success, `429` rate limit, `500` server error.
- The web UI exports **PNG** only ("We are yet to support other file formats").

The app's own "Copy as cURL" button builds the request like this (from the page's JS bundle):

```js
const body = { name: template.name, params: template.params, canvas: template.canvas };
// curl -H "Content-Type: application/json" <BASE>/v1/images -d '<body>'
```

**Templates and fields.** These come from the page's JS bundle. The fields change per template, so copy the exact JSON with the site's *API Request → Copy as JSON* button before you code against it.

| Template `name` | Fields seen in the renderer and editor |
|---|---|
| `9gag` | `title` (`text`, `bgColor`, text colour), `background` (`url`), `position` (`top` / `bottom`) |
| `pubity` | `title`, `logo`, `background`, position (`top-left`, `top-right`, `none`) |
| `thetatvaindia` | `title`, `description`, `highlightedText`, `logo`, `background`, `position`, `socialMedia` |
| `tdtinsta` | title, username, full or short name, carousel flag, "SWIPE" marker, vignette |

Shared editor controls: font family, font size, font weight, alignment, text colour, background colour, background image URL. The default canvas is **1080 × 1080**.

> **Brand note.** The templates copy the look of real third-party Instagram pages (9GAG, Pubity, and others). Before posting for the Osmossi brand, decide whether that look is acceptable. Our own templates may be the better choice.

## 1.3 Publishing: Instagram Graph API

Sources: <https://developers.facebook.com/docs/instagram-platform/content-publishing/>, <https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login/business-login>.

Requirements:

- An Instagram **professional account** (Business or Creator).
- A Meta app using **Business Login for Instagram**, with scopes `instagram_business_basic` and `instagram_business_content_publish`. Accounts you own and add to the app need only Standard Access. Serving other people's accounts needs Advanced Access, which requires App Review.
- OAuth: `https://www.instagram.com/oauth/authorize` → exchange the code at `https://api.instagram.com/oauth/access_token` → get a **long-lived token (60 days)**. Refresh it with `https://graph.instagram.com/refresh_access_token`.

Flow (each call runs in a Netlify Function and never in the browser):

```
1. POST /<IG_ID>/media           image_url=<PUBLIC JPEG URL>&caption=...   -> { id: <container> }
2. GET  /<container>?fields=status_code   until FINISHED  (EXPIRED | ERROR | FINISHED | IN_PROGRESS | PUBLISHED)
3. POST /<IG_ID>/media_publish   creation_id=<container>                     -> { id: <media> }
```

- **JPEG only.** Meta fetches the image from a **publicly reachable URL** at publish time.
- Carousels: build one container per image, then `media_type=CAROUSEL` with `children` (up to 20 IDs), then publish.
- Limit: **100 API-published posts per moving 24 h**; a carousel counts as one. Check usage with `GET /<IG_ID>/content_publishing_limit`.
- Hosts: `graph.instagram.com` (Instagram Login) or `graph.facebook.com` (Facebook Login).

**The format mismatch to solve.** The generator returns PNG, and Instagram accepts only JPEG at a public URL. The pipeline is therefore:

```
generator (PNG) -> Netlify Function converts to JPEG (e.g. `sharp`) -> Netlify Blobs
  -> served by a public function route /media/<random-id>.jpg -> Instagram fetches it -> delete it after publishing
```

Use unguessable IDs and delete each image after publishing (or after 24 h).

**Token storage.** The long-lived token changes every time it is refreshed. A function cannot rewrite its own env vars, so keep the current token in Netlify Blobs, encrypted with a key held in an env var (`IG_TOKEN_ENC_KEY`). A Netlify **scheduled function** refreshes it, for example weekly. Note that scheduled functions are limited to 30 s.

**Scheduling posts.** Store queued posts in Blobs. Have a scheduled function publish the posts that are due. The repo also has the Osmossi ERP social calendar (posts and statuses), which could be the source of truth for the queue. That is a later integration.

## 1.4 Scoring: "beach day" vs. post readiness

The brief asks how to score a *"beach day"* from the API fields. The generator's fields (title, background, colours, logo, position, canvas) say nothing about weather or beaches. That line seems to come from a different project. **What we propose instead** is a **Post Readiness Score (0–100)**, computed in the browser and repeated on the server before publishing. Every input comes from the API payload or from Instagram's limits:

| Check | Field(s) | Points | Source of rule |
|---|---|---|---|
| Image will render (template known, background URL loads, HTTPS) | `name`, `background.url` | 20 (hard gate) | Generator docs |
| Text is readable: contrast ratio of text colour vs `bgColor` ≥ 4.5:1 (full points), ≥ 3:1 (half) | `title.color`, `title.bgColor` | 20 | WCAG 2.2 SC 1.4.3 contrast formula |
| Title length fits (e.g. ≤ 90 chars; fewer lines at the chosen font size) | `title.text`, font size | 15 | Heuristic; tune by looking at previews |
| Canvas is a supported feed aspect ratio (1:1 default; 4:5 portrait) | `canvas` | 15 (hard gate) | Instagram media reference: confirm the exact range there |
| Caption within limits (≤ 2,200 chars, ≤ 30 hashtags, ≤ 20 @mentions) | caption | 15 (hard gate) | Instagram IG User Media reference: confirm |
| Brand elements present (logo / handle) when the template supports them | `logo`, `username` | 10 | Brand rule |
| Publishing quota left today | `content_publishing_limit` | 5 (hard gate when 0) | Content Publishing docs |

A hard gate blocks **Publish** and shows an accessible error message. For readability, the score appears as a number and a label (e.g. "Ready", "Needs work"), not by colour alone.

## 1.5 Open questions

- Is there an official MCP server for this generator at all? If one exists, the Agent Console could call it through the Claude API MCP connector (`mcp_servers` + `mcp_toolset`, beta `mcp-client-2025-11-20`). Until then, call the REST endpoint from a function.
- The generator's IP-based rate limit is shared by every caller on Netlify's egress IPs. Cache rendered images by a hash of the payload, and handle `429`.
- The generator is a free third-party service with no SLA. Plan a fallback: render with our own `satori` / `@vercel/og`-style function. The generator itself uses Satori; the preview SVG shows it.
