# MEMORY — read me first, every session

Repo: `z-ey3eiei/my-app.site` (name it "my-app.site") · Cloudflare Worker `platform-onenext`
Last updated: 2026-11-02

## How to use this file (agent rules)
1. Read `MEMORY.md` + `memory/decisions.log` **before** any work.
2. On every meaningful change, append one line to `memory/decisions.log` in the same commit.
3. Update the STATE table whenever something ships or breaks. Never rewrite history in the log.
4. Never write secrets here. Tokens are device-flow only (`gh auth refresh`, `wrangler login`) — env vars set in the Vercel project UI are NOT visible to the sandbox.

## STATE
| item | status | evidence |
|---|---|---|
| Web app (Vite 7 + React 19 + TS + Tailwind v4) | builds | `npm run build` → `dist/` |
| Lint / typecheck / tests (web) | green | `npm run check` 0 errors, 8 tests |
| SDK `@platform-onenext/core` + `@platform-onenext/database` | green | `cd sdk && npx vitest run` → 82 tests, coverage 91.9% stmts |
| Cloudflare Worker + static assets | **deployed** | `https://platform-onenext.mysterious-draw.workers.dev` |
| — `/v1/status` | 200 JSON | `{"ok":true,"runtime":"cloudflare-workers"}` |
| — `/` SPA + deep links | 200 | `/docs/pricing` → 200 (SPA fallback) |
| — CORS preflight `/v1/*` | 204 + allow-* | verified with curl |
| — `/v1/users` etc | **404 code 1042, NOT 501** | open bug, see NEXT |
| GitHub Actions workflows | blocked | token lacks `workflow` scope; files live in old repo only |
| Vercel (`platform-onenext`) | superseded | Hobby plan bans commercial use → do not build on it |

## Deploy
```bash
cd cloudflare && npx wrangler deploy        # needs `wrangler login --device` (5-min code, user approves)
npx wrangler deploy --temporary             # no login; URL is claimable for 60 min
```
`wrangler.jsonc` serves `../dist` with `not_found_handling: single-page-application` and
`run_worker_first: ["/v1/*"]`. Build the web app first: `npm run build`.

## NEXT (priority order)
1. Fix `/v1/*` 404: `run_worker_first: ["/v1/*"]` did not route `/v1/users` to the Worker (got `error code: 1042`). Try `run_worker_first: true` and re-test.
2. Rebuild `src/App.tsx` from branch `starter` of the old repo and deploy the full site (live site is still the 9-file minimal page).
3. Phase 2 API in the Worker: `/v1/auth/*`, `/v1/db/query` — must keep the `{ data, error }` shape the SDK already expects.
4. Realtime later = **SQLite-backed Durable Object only** (KV-backed DO namespaces cannot be created on Free, and block downgrade).

## HARD CONSTRAINTS (do not re-litigate)
- No `workflow` scope → never put `.github/workflows/*` in a push, the push is rejected wholesale.
- Vercel Hobby: no Stripe/billing/commercial traffic.
- Cloudflare Free limits: 100k req/day, 10ms CPU/req, 128MB, 50 subrequests, 20k asset files.
- Do not upgrade to Workers Paid ($5/mo) before there is real traffic.
- DO NOT run `git push --force` on `main` of this repo.
