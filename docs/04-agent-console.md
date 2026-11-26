# 4. Agent Console

A password-protected page inside the deployed PWA where the user can:

1. See every agent in `.claude/agents/` with its **name, model, and personality**.
2. **Chat** with any agent. Claude is called only from a Netlify Function, with the key in an env var.
3. Review **code changes an agent proposes**. They are committed to GitHub **only after the user clicks Approve**. The push starts a Netlify redeploy.

Sources: Claude API docs (Messages, streaming, tool use, models), served through the `claude-api` skill on 2026-11-26 and checked against <https://platform.claude.com/docs>; GitHub REST *Repository contents* <https://docs.github.com/en/rest/repos/contents>, *Git trees* <https://docs.github.com/en/rest/git/trees>, *Git commits / refs* <https://docs.github.com/en/rest/git>, *Managing personal access tokens* <https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens>; Netlify docs listed in doc 3; Claude Code subagent file format <https://code.claude.com/docs/en/sub-agents>.

## 4.1 Listing agents

Claude Code subagents are Markdown files with YAML frontmatter. The standard keys are `name`, `description`, `tools`, and `model` (`opus` / `sonnet` / `haiku` / `inherit` or a full ID). The body is the system prompt. **"Personality" is not a standard key.** We propose an optional `personality:` key (one line, shown in the UI). If it is missing, the UI shows the first paragraph of the body.

```markdown
---
name: pwa-expert
description: Builds and audits the PWA shell, manifest and service worker.
model: opus
personality: Precise, test-first, explains trade-offs briefly.
tools: Read, Edit, Grep
---
You are the PWA expert for OsmossiExperts…
```

At build time, `scripts/build-agents.mjs` parses these files into `public/agents.json` (name, description, model, personality). The system prompts go to a **server-only** bundle, imported by the chat function, so the browser never downloads them. Adding an agent therefore needs a redeploy, which the approve flow already triggers.

Model alias → Claude API ID (current as of 2026-11-26):

| Alias in file | API model ID |
|---|---|
| `opus` / `inherit` / missing | `claude-opus-5-5` |
| `sonnet` | `claude-sonnet-5-5` |
| `haiku` | `claude-haiku-5-5` |
| full ID | used as is, but only if it is in an allow-list of model IDs |

## 4.2 Chat (short turns, streamed)

`POST /api/agents/chat { agent, messages }` → the function checks the session, loads that agent's system prompt from the server bundle, and streams the reply.

```ts
import Anthropic from "@anthropic-ai/sdk";
const client = new Anthropic({ apiKey: Netlify.env.get("ANTHROPIC_API_KEY") });

const stream = client.beta.messages.stream({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  system: agent.systemPrompt,
  messages,                                   // the client sends history; the server caps its length
  output_config: { effort: "medium" },         // Opus 5.5 default is medium; set it explicitly
  tools: [proposeChangeTool],                  // see 4.3
  betas: ["server-side-fallback-2026-07-01"],
  fallbacks: "default",                        // on a policy refusal the API retries on a fallback model
});
// Re-emit text deltas to the browser as SSE; on finalMessage(), store any proposal.
```

Notes from the Claude API docs:

- Thinking on Opus 5.5 is adaptive and **cannot be disabled**. Control cost with `output_config.effort` (`low` for chit-chat, `high` for code work).
- Always check `stop_reason`: `refusal` (with `stop_details`), `max_tokens`, `tool_use`.
- Forced `tool_choice` (`any` / `tool`) returns **400** on Opus 5.5. Use `auto`, `strict: true` on the tool, and a prompt instruction.
- With streaming and client tools, set `eager_input_streaming: true` and **validate the parsed tool input against its schema** before using it.
- Use prompt caching on the agent's long system prompt (`cache_control`). It makes repeat turns cheaper.
- **The 60 s cap:** a long reply can hit Netlify's streaming limit. When a turn needs a tool loop or a large code change, the chat function returns `{ jobId }` and hands off to the background job (4.5).

## 4.3 Proposing changes: a tool, never a direct write

The agent gets **one tool**. It can only *propose* a change. Nothing is written to GitHub at this step.

```ts
const proposeChangeTool = {
  name: "propose_change",
  description: "Propose edits to files in this repository. A human must approve before anything is committed.",
  strict: true,
  eager_input_streaming: true,
  input_schema: {
    type: "object",
    additionalProperties: false,
    required: ["summary", "files"],
    properties: {
      summary: { type: "string" },
      files: {
        type: "array",
        items: {
          type: "object",
          additionalProperties: false,
          required: ["path", "content"],
          properties: { path: { type: "string" }, content: { type: "string" } },
        },
      },
    },
  },
};
```

To read code, give the agent a `read_file` tool that the server answers from GitHub (`GET /repos/{o}/{r}/contents/{path}`, same allow-list, files ≤ 1 MB). The function **validates** the proposal (4.6) and stores it in the `proposals` Blob store as `pending`. For each file it records the `baseSha` of the version it read. The tool result sent back to Claude is just "Proposal P-123 saved for human review".

The UI shows a **diff per file**, the summary, and the agent's name, with **Approve** and **Reject** buttons.

## 4.4 Approve → commit → redeploy

`POST /api/proposals/:id/approve` (session + CSRF checks) runs **one atomic commit** through the Git Data API. The contents API (`PUT /contents/{path}`) writes one file per commit, and GitHub warns that it conflicts with concurrent deletes, so it is only good for single-file changes.

```
1. GET  /repos/{o}/{r}/git/ref/heads/{branch}            -> headSha
2. GET  /repos/{o}/{r}/git/commits/{headSha}             -> baseTreeSha
   (re-check: each file's current blob sha == proposal.baseSha; if not -> 409 "stale, re-run")
3. POST /repos/{o}/{r}/git/trees   { base_tree: baseTreeSha, tree: [{ path, mode: "100644", type: "blob", content }] }
4. POST /repos/{o}/{r}/git/commits { message: "agent(<name>): <summary>\n\nProposal P-123, approved by <user>", tree, parents: [headSha] }
5. PATCH /repos/{o}/{r}/git/refs/heads/{branch} { sha: newCommit, force: false }   // fails if branch moved -> retry from 1
```

Headers: `Authorization: Bearer $GITHUB_TOKEN`, `Accept: application/vnd.github+json`, `X-GitHub-Api-Version: 2026-03-10` (the version the docs show today).

Netlify's Git integration builds on every push to the production branch. The **redeploy needs no extra call** (a Netlify *build hook* is only needed if auto-deploys are turned off). After approval, the UI polls the commit and shows "Deploying…". The service worker's update toast (doc 2) then loads the new version.

**Recommendation:** commit to a branch `agents/P-123` and open a PR (Pull requests: write). Netlify builds a **Deploy Preview** for it, and the user merges after checking the preview. Direct-to-`main` is simpler but deploys straight to production. The owner decides; the code supports both through `GITHUB_BRANCH`.

## 4.5 Long agent jobs within Netlify's limits

| Need | Mechanism |
|---|---|
| Reply ≤ ~50 s | Streaming function (60 s cap, 20 MB) |
| Multi-step tool loop or big change (minutes) | **Background function** (15 min). The caller gets 202 with a `jobId` |
| > 15 min | Split into checkpoints: the job saves its conversation state to Blobs and starts a follow-up background invocation, or move to Claude **Managed Agents** (Anthropic hosts the loop and sandbox; beta; bills session runtime on top of tokens) |

Job flow:

```
POST /api/agents/jobs  -> creates job {status: queued} in Blobs (strong), calls the background fn with {jobId}
background fn          -> loop: messages.stream(...) → run read_file/propose_change → append progress events to the job record
                          stops well before 15 min (e.g. at 13 min) → status "paused" with saved messages, or "done"
GET  /api/agents/jobs/:id  -> UI polls every 2–3 s (or long-polls ≤ 25 s) and renders progress events
```

Guards: a per-job cap on tokens and on tool iterations (task budgets, beta, are an option). One running job per user. The job record stores `usage` so it can be charged against daily budgets.

## 4.6 Security model

| Threat | Control |
|---|---|
| Strangers use the Console or burn API credit | **Password gate.** `POST /api/login` checks against `CONSOLE_PASSWORD_HASH` (Node `crypto.scrypt`, `timingSafeEqual`). It sets a session cookie `HttpOnly; Secure; SameSite=Strict; Path=/api; Max-Age=8h`, HMAC-signed with `SESSION_SECRET` and recorded in the `sessions` store so it can be revoked. Login is rate limited (e.g. 5 per 15 min per IP) and allows paste and password managers (WCAG 3.3.8). |
| Secrets leak to the browser | Keys exist only in function env vars (Functions scope, marked secret). The browser bundle never contains them. Agent system prompts are server-only. Responses never echo env or headers. |
| Token scope too broad | **Fine-grained PAT**: resource owner = repo owner; **Only select repositories → this repo**; permissions **Contents: read and write** (Metadata: read is added automatically; add Pull requests: write only for the PR flow). **No Workflows permission**, so agents cannot edit `.github/workflows`. Set an expiration (e.g. 90 days, with a reminder to rotate). A GitHub App with the same rights is a stronger alternative later. |
| Agent writes where it shouldn't (including prompt injection from file contents or chat) | **Server-side path allow-list**, checked at proposal time *and again* at approval. Normalize the path (reject `..`, absolute paths, backslashes, NUL, symlinks). Allow e.g. `src/**`, `public/**` (except `public/sw.js` and `public/manifest.webmanifest`: a bad service worker can take over every page and keep serving after the bug is fixed), `docs/**`, `.claude/agents/*.md`. **Deny** `netlify/functions/**`, `netlify.toml`, `package.json` / lockfiles, `scripts/**`, `.github/**`, `.env*`. An agent must never be able to change the code that enforces these rules. Limits per proposal: ≤ 20 files, ≤ 200 KB each, text only. A human reviews the diff before anything is committed. |
| Cross-site requests trigger Approve | `SameSite=Strict` plus an `Origin` check on every POST, plus a per-session CSRF token header |
| Abuse / cost spikes | Netlify `rateLimit` in each function's config (per IP, window ≤ 180 s, returns 429). **Daily budgets** in Blobs: Claude tokens per user, jobs per day, commits per day. A max `max_tokens` per request. Return a clear message when a limit is hit. |
| Model output treated as trusted | Render chat as text or sanitized Markdown (no raw HTML). Diffs are shown, never executed. Tool inputs are validated against the schema. |
| Audit | Every proposal and decision is stored (who, when, which commit). The commit message names the agent and the proposal. |

## 4.7 Open questions

- Branch + PR (recommended) or direct to `main`?
- One shared password, or per-partner accounts?
- Should agents be able to edit `.claude/agents/*.md`, that is, other agents' prompts? Proposed: yes, but always shown as a separate, highlighted diff.
