# AGENTS.md

Astro 7 + Tailwind CSS v4 starter. Full conventions in [CLAUDE.md](CLAUDE.md) — read it before starting any work.

## Dev Server (AI agents)

When you need to start the dev server, always use background mode:

```sh
pnpm exec astro dev --background   # pnpm (default)
npx astro dev --background         # npm
yarn astro dev --background        # yarn
bunx astro dev --background        # bun
```

Manage with: `astro dev stop` · `astro dev status` · `astro dev logs`
