# my-app.site

Full-stack platform: Vite/React site + Cloudflare Worker API + a typed SDK, one repo, one origin.

```text
src/         web app (Vite 7, React 19, TS, Tailwind v4)
cloudflare/  Worker that serves dist/ as assets and owns /v1/*
sdk/         @platform-onenext/core (HTTP client) + @platform-onenext/database (query builder)
MEMORY.md    agent + human working memory — read first
memory/      append-only decision log
```

## Start here
```bash
npm install && npm run build            # web → dist/
cd sdk && npm install && npx vitest run # 82 tests
cd ../cloudflare && npx wrangler deploy # needs `wrangler login --device`
```

## Live
- Site + API: https://platform-onenext.mysterious-draw.workers.dev (`/v1/status`)
- This URL is on a temporary preview account until claimed: https://dash.cloudflare.com/claim-preview
