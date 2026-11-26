# 3. Architecture: static Netlify site + Netlify Functions

Sources: Netlify Functions overview <https://docs.netlify.com/build/functions/overview/>, Functions configuration (limits) <https://docs.netlify.com/build/functions/configuration/>, Functions API reference (streaming, `Config`) <https://docs.netlify.com/build/functions/api/>, Background Functions <https://docs.netlify.com/build/functions/background-functions/>, Netlify Blobs <https://docs.netlify.com/build/data-and-storage/netlify-blobs/>, Environment variables <https://docs.netlify.com/build/environment-variables/overview/>, Rate limiting <https://docs.netlify.com/manage/security/secure-access-to-sites/rate-limiting/>.

## 3.1 Shape

```
Phone (installed PWA)
  │  static files from Netlify CDN (HTML/CSS/JS, manifest, sw.js, agents.json)
  │  fetch('/api/...') with an HttpOnly session cookie
  ▼
Netlify Functions (Node, TypeScript)  ── secrets only here ──►  Claude API
  │                                                             Instagram Graph API
  │                                                             GitHub REST API
  │                                                             Posts Generator API
  ▼
Netlify Blobs (site-wide stores): sessions, jobs, proposals, media, ig-token, rate counters
```

The browser holds **no secrets**. Every third-party call goes through a function.

## 3.2 Repository layout (proposed)

```
/                      static site (Vite build → dist/)
  index.html, src/…, public/manifest.webmanifest, public/sw.js, public/icons/…
netlify/functions/
  auth-login.mts       POST /api/login          (password gate)
  agents-chat.mts      POST /api/agents/chat    (streamed Claude reply, ≤60 s)
  agent-job-background.mts  POST /api/agents/jobs  (long job, ≤15 min, returns 202)
  agent-job-status.mts GET  /api/agents/jobs/:id
  proposals.mts        GET/POST /api/proposals[/:id/approve|reject]
  ig-render.mts        POST /api/ig/render      (generator → PNG → JPEG → Blobs)
  ig-publish.mts       POST /api/ig/publish     (container → status → publish)
  media.mts            GET  /media/:id.jpg      (public, for Instagram to fetch)
  ig-token-refresh.mts scheduled (weekly)
  ig-scheduler.mts     scheduled (every 5–15 min) publishes queued posts
scripts/build-agents.mjs   parses .claude/agents/*.md → public/agents.json at build time
.claude/agents/*.md        the expert agents
netlify.toml
```

## 3.3 Function basics (modern signature)

```ts
// netlify/functions/agents-chat.mts
import type { Config, Context } from "@netlify/functions";

export default async (req: Request, context: Context) => {
  const key = Netlify.env.get("ANTHROPIC_API_KEY"); // process.env also works
  // … return new Response(readableStream) to stream
};

export const config: Config = {
  path: "/api/agents/chat",
  rateLimit: { windowLimit: 20, windowSize: 60, aggregateBy: ["ip", "domain"] },
};
```

- Default functions directory: `netlify/functions`.
- `config.path` sets the URL, so no redirects are needed.
- `rateLimit` is set **in code only**. It cannot go in `netlify.toml`, and `windowSize` can be at most 180 s.
- Background function: `export const config = { background: true, path: "/api/agents/jobs" }`. The older `-background` filename suffix still works. The caller gets **202** at once.

## 3.4 Limits that drive the design

| Setting | Value | Consequence |
|---|---|---|
| Synchronous execution | **60 s** (not configurable) | Short chat turns only |
| Streaming function | **60 s**, response **≤ 20 MB** | Stream Claude tokens to the UI; a long agent turn can still be cut off |
| Background execution | **15 min** | Long agent jobs, multi-step tool loops |
| Scheduled execution | **30 s** | Token refresh and a small publish queue; hand off to a background function if needed |
| Buffered request/response payload | **6 MB** | Fine for JSON. Keep generated images in Blobs and pass only an ID |
| Background request payload | **256 KB** | Send a job ID and a short prompt, not whole files |
| Memory | 1024 MB default | `sharp` JPEG conversion fits |
| Region | `cmh` (US East, Ohio) default | Pin Blobs and functions close together |

## 3.5 Netlify Blobs

```ts
import { getStore } from "@netlify/blobs";
const jobs = getStore({ name: "agent-jobs", consistency: "strong" });
await jobs.setJSON(jobId, { status: "running", events: [] });
const job = await jobs.get(jobId, { type: "json" });
```

- **Site-wide stores** (`getStore`) last across deploys. That matters because each approval redeploys the site. Do **not** use deploy-scoped stores (`getDeployStore`) for jobs or proposals.
- Consistency: eventual by default (changes reach every edge within 60 s). Use **`strong`** for job status, proposals, sessions, and rate counters.
- Limits: store names ≤ 64 bytes with no `/` or `:`; keys ≤ 600 bytes, not starting with `/`; objects ≤ 5 GB; metadata ≤ 2 KB; `list()` pages of 1,000.

| Store | Key | Value |
|---|---|---|
| `sessions` | session ID | `{ createdAt, expiresAt, ipHash }` |
| `agent-jobs` | job ID | status, progress events, final message, token usage |
| `proposals` | proposal ID | agent, files[{path, newContent, baseSha}], summary, status, decidedAt |
| `media` | random ID | JPEG bytes (deleted after publishing) |
| `ig` | `token` | encrypted long-lived token + expiry |
| `ratelimit` | `user:<id>:<yyyy-mm-dd>` | daily counters (Claude tokens, commits) |

## 3.6 Environment variables

Set them in the Netlify UI or CLI, with **scope: Functions**, and mark each one **"Contains secret values"** (Secrets Controller). Use separate values per deploy context (Production / Deploy Previews / Branch deploys / Local development; local-dev values cannot be marked secret).

| Variable | Purpose |
|---|---|
| `ANTHROPIC_API_KEY` | Claude API (functions only) |
| `GITHUB_TOKEN` | Fine-grained PAT for this repo only (doc 4) |
| `GITHUB_OWNER`, `GITHUB_REPO`, `GITHUB_BRANCH` | Commit target; not secrets |
| `CONSOLE_PASSWORD_HASH` | scrypt hash of the Agent Console password |
| `SESSION_SECRET` | HMAC key for session cookies |
| `IG_APP_ID`, `IG_APP_SECRET`, `IG_TOKEN_ENC_KEY` | Instagram OAuth + token encryption |

Never prefix these with a bundler's public prefix (`VITE_…`). Those values get built into the browser bundle.

## 3.7 netlify.toml (sketch)

```toml
[build]
  command = "node scripts/build-agents.mjs && npm run build"
  publish = "dist"

[functions]
  directory = "netlify/functions"
  node_bundler = "esbuild"

[[headers]]
  for = "/*"
  [headers.values]
    Content-Security-Policy = "default-src 'self'; img-src 'self' data: blob: https:; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"
    Permissions-Policy = "camera=(), microphone=(), geolocation=()"

[[headers]]
  for = "/sw.js"
  [headers.values]
    Cache-Control = "no-cache"
```

## 3.8 Open questions

- Which frontend stack? Plain Vite + TypeScript is the lightest option, and any of them works with this layout.
- Will one person use the Console (one password), or several partners (per-user passwords, or Netlify Identity / an OAuth provider)?
