# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Detailed Tailwind rules live in `.claude/rules/tailwind.md` and are loaded automatically.

## Design Reference

If `DESIGN.md` exists in the project root, read it before starting any UI or feature work. It contains the site's design intent, color palette, typography, layout guidelines, and component conventions — treat it as the source of truth for visual and UX decisions.

## Commands

Replace `<pm>` with your package manager (`npm`, `yarn`, `pnpm`, `bun`, etc.).

```sh
<pm> run dev           # Start dev server at localhost:4321 (for humans)
<pm> run build         # Build production site to ./dist/
<pm> run preview       # Preview production build locally
<pm> run astro check   # Type-check .astro files
<pm> run check         # Biome check --write (lint + format combined)
<pm> run lint          # Biome lint --write
<pm> run format        # Biome format --write
<pm> run astro-upgrade # Upgrade Astro via @astrojs/upgrade
```

**When you (as an AI agent) need to start the dev server** — whether acting autonomously or at the user's request — always use background mode to avoid blocking your terminal:

```sh
pnpm exec astro dev --background   # pnpm (default)
npx astro dev --background         # npm
yarn astro dev --background        # yarn
bunx astro dev --background        # bun
```

Manage the background server with: `astro dev stop` · `astro dev status` · `astro dev logs`

## Astro Docs MCP (`mcp__AstroDocs__search_astro_docs`)

Use this tool over training-data knowledge for Astro-specific APIs, integrations, and configuration: component syntax, frontmatter, slots, Content Collections, Actions, View Transitions, SSR/SSG config, `astro.config.mjs` options, and any Astro 7 feature that may differ from earlier versions. Always check the docs when behavior is uncertain or the user hits an unexpected error.

## Stack

- **Astro 7** — file-based routing under `src/pages/`; `.astro` components use frontmatter (`---`) for server-side logic
- **Tailwind CSS v4** — via the `@tailwindcss/vite` plugin (no `tailwind.config.*`); imported in `src/styles/global.css`
- **TypeScript** — strict mode via `astro/tsconfigs/strict`
- **pnpm** (default) — package manager (node ≥ 22.12.0 required); npm/yarn/bun also work

## Architecture

- `src/layouts/Layout.astro` — root HTML shell; import global CSS here
- `src/pages/` — each `.astro` file becomes a route automatically
- `src/components/` — reusable `.astro` components
- `src/assets/` — static assets processed by Vite (use `import` to reference)
- `public/` — assets served as-is (favicon, etc.)

## Package Security

`.npmrc` enforces one project-wide constraint:

- `min-release-age=3` — blocks packages published fewer than 3 days ago

Build script permissions are managed explicitly via `allowBuilds` in `pnpm-workspace.yaml` and `allowScripts` in `package.json` (e.g. `esbuild`, `sharp`, `fsevents`). Only listed packages may run install scripts.

**When adding a new dependency:** if it fails to install due to these constraints, add it to `allowBuilds` in `pnpm-workspace.yaml` (if it needs build scripts) or `minimumReleaseAgeExclude` (if it was just published).

## Code Quality (Biome)

No ESLint or Prettier. JS/TS/JSON/CSS is managed by **Biome**. `.astro` files are excluded from Biome — use `<pm> run astro check` for type-checking them.

- Single quotes, semicolons `asNeeded`, trailing commas; JSX attributes use double quotes
- 80-char line width, 2-space indent, LF line endings (`.editorconfig`)
- Always run `<pm> run check` before finishing any code change

## Icons

`@lucide/astro` provides tree-shakable SVG icons as Astro components:

```astro
---
import { Camera } from '@lucide/astro';
---

<Camera size={16} stroke-width={1} class="text-muted" />
```

Props: `size` (default `24`), `color` (default `currentColor`), `stroke-width` (default `2`), `absoluteStrokeWidth`. Tailwind classes apply via the `class` prop.

When a framework integration is added, install and use the matching Lucide package inside that framework's components:

| Integration | Package | Use in |
|---|---|---|
| `@astrojs/react` | `lucide-react` | React components (`.tsx`); may also unify `.astro` imports |
| `@astrojs/vue` | `@lucide/vue` | Vue components (`.vue`) |
| `@astrojs/svelte` | `@lucide/svelte` | Svelte components (`.svelte`) |

Never import `@lucide/astro` inside React/Vue/Svelte components — it only works in `.astro` files.
