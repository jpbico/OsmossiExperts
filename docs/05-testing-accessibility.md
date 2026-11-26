# 5. Testing and accessibility

Sources: WCAG 2.2 *What's new* <https://www.w3.org/WAI/standards-guidelines/wcag/new-in-22/>, WCAG 2.2 quick reference <https://www.w3.org/WAI/WCAG22/quickref/>, MDN PWA guides (doc 2), Netlify Functions docs (doc 3), Playwright <https://playwright.dev/>, axe-core <https://github.com/dequelabs/axe-core>.

## 5.1 Test pyramid

| Layer | Tool | What to cover |
|---|---|---|
| Unit | **Vitest** | Post-readiness score (contrast maths, caption limits, gates); path allow-list and normalization (`../`, `%2e%2e`, absolute paths, symlinks, deny-list wins over allow-list); frontmatter parser for `.claude/agents`; session cookie sign/verify; rate-limit counters |
| Function / integration | Vitest + **MSW** (or `undici` MockAgent), run against the function handlers | Claude stream handling (`end_turn`, `tool_use`, `refusal`, `max_tokens`); `propose_change` input validation; the Git Data commit sequence, including **stale `baseSha` → 409** and **ref update rejected → retry**; Instagram container → `IN_PROGRESS` → `FINISHED` → publish; `ERROR` / `EXPIRED`; quota at 0; generator `429` |
| Local end-to-end | `netlify dev` + **Playwright** | Login → chat → proposal → approve (against a **test repo** and a test token, never the real repo); render → publish against a mocked Graph API |
| PWA | Playwright + Chrome DevTools / Lighthouse | Manifest valid (192 and 512 icons, `display: standalone`); SW registered; offline shell; update toast after a new SW is installed |
| Accessibility | **@axe-core/playwright** on every page in CI, plus manual screen-reader passes | See 5.3 |
| Real devices | Manual, before each release | iPhone (Safari: Add to Home Screen, standalone, safe areas, push if used); Android Chrome (WebAPK install) |
| Deploy Previews | Netlify Deploy Preview per PR | Run the Playwright smoke suite against the preview URL |

No test may call the live Claude API, GitHub, or Instagram by default. Allow live smoke tests only behind an explicit flag, with test accounts and low budgets.

## 5.2 Security tests (must pass before the Console ships)

- Every `/api/*` route except `/api/login` and `/media/*` returns 401 without a valid session.
- A POST with a foreign `Origin` header, or without the CSRF token, returns 403.
- A proposal touching `netlify/functions/x.mts`, `netlify.toml`, `.github/workflows/x.yml`, `package.json`, or `../secrets` is rejected at save time **and** at approve time, even if someone edits the Blob record directly.
- `grep` of the built `dist/` finds no `sk-ant`, `github_pat_`, IG tokens, or agent system prompts.
- Login rate limiting kicks in after N attempts.
- The `rateLimit` config is present on every function that costs money.

## 5.3 Accessibility target: WCAG 2.2 AA

Criteria new in WCAG 2.2 that matter most in this app:

| SC | Level | In this app |
|---|---|---|
| 2.4.11 Focus Not Obscured (Minimum) | AA | Sticky bottom nav, install banner, and toasts must not hide the focused element |
| 2.5.7 Dragging Movements | AA | Template carousel: offer Previous/Next buttons as well as swipe |
| 2.5.8 Target Size (Minimum) | AA | Touch targets ≥ 24×24 CSS px (aim for 44×44 on mobile) |
| 3.2.6 Consistent Help | A | Help link in the same place on every screen |
| 3.3.7 Redundant Entry | A | Remember template fields and caption between steps |
| 3.3.8 Accessible Authentication (Minimum) | AA | Console password field allows paste and password managers; no CAPTCHA puzzles |

Also, from the existing criteria:

- **Contrast**: 1.4.3 (4.5:1 text) and 1.4.11 (3:1 UI). The post-readiness score reuses the same contrast check for the *generated image* text.
- **Alt text**: preview images carry alt text from the post title. Prompt the user for Instagram alt text too.
- **Live regions**: the streaming chat goes in an `aria-live="polite"` region, announced per completed message rather than per token. Job progress and "Deploying…" status use `role="status"`.
- **Diff view**: use real table or list semantics, mark added and removed lines with text (`+`, `−`, or visually hidden "added"/"removed"), not colour alone, and make it keyboard navigable.
- **Motion and zoom**: respect `prefers-reduced-motion`; the layout works at 200% zoom and 320 px width (1.4.10 Reflow); never block pinch-zoom in the viewport meta.
- **Forms**: every field has a visible `<label>`; errors are announced (`aria-describedby` + `aria-invalid`); score gates state *why* Publish is disabled.

Manual checks each release: VoiceOver on iOS (installed app), TalkBack on Android, keyboard-only on desktop.

## 5.4 CI (GitHub Actions, proposed)

```
on: pull_request
jobs: lint → typecheck → vitest → build → playwright (+axe) against `netlify dev` or the Deploy Preview URL
```

Agents cannot change this pipeline: their GitHub token has no Workflows permission, and `.github/**` is on the deny-list.
