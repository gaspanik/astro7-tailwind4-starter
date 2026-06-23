# Copilot Instructions

This project is an **Astro 7 + Tailwind CSS v4** starter. Full conventions are in [CLAUDE.md](../CLAUDE.md) — read it before starting any work.

## Design Reference

If `DESIGN.md` exists in the project root, read it before starting any UI or feature work. It contains the site's design intent, color palette, typography, layout guidelines, and component conventions — treat it as the source of truth for visual and UX decisions.

## MCP Tools

### Astro Docs MCP (`mcp_astro_docs_se_search_astro_docs`)

Use this tool when working with Astro-specific APIs, integrations, or configuration. Prefer it over training-data knowledge for:

- Astro component syntax, frontmatter, slots, `<slot />` usage
- Integrations: `@astrojs/react`, `@astrojs/tailwind`, `@astrojs/image`, etc.
- Content Collections, Actions, View Transitions, SSR/SSG config
- `astro.config.mjs` options and Vite plugin setup
- Any Astro 7 feature that may differ from earlier versions

Always check the docs when the behavior is uncertain or the user hits an unexpected error.

## Stack

- **Astro 7** — file-based routing under `src/pages/`; `.astro` components use frontmatter (`---`) for server-side logic
- **Tailwind CSS v4** — no `tailwind.config.*`; configured via `@theme` in `src/styles/global.css`; imported with `@import "tailwindcss"`
- **TypeScript** — strict mode (`astro/tsconfigs/strict`)
- **pnpm** (default) — package manager (Node ≥ 22.12.0); npm/yarn/bun also work

## Project Structure

```
src/layouts/Layout.astro   # Root HTML shell — import global CSS here
src/pages/                 # Each .astro file becomes a route
src/components/            # Reusable .astro components
src/assets/                # Static assets processed by Vite (use import)
public/                    # Assets served as-is
src/styles/global.css      # Tailwind entry — @theme tokens go here
```

## Dev Commands

> Replace `<pm>` with your package manager (`npm`, `yarn`, `pnpm`, `bun`, etc.).

```sh
<pm> run dev          # localhost:4321 (for humans)
<pm> run build        # Build to ./dist/
<pm> run preview      # Preview production build
<pm> run astro check  # Type-check .astro files (separate from Biome)
<pm> run check        # Biome lint + format (run before finishing any change)
```

**When you (as an AI agent) need to start the dev server** — whether acting autonomously or at the user's request — always use background mode to avoid blocking your terminal:

```sh
pnpm exec astro dev --background   # pnpm (default)
npx astro dev --background         # npm
yarn astro dev --background        # yarn
bunx astro dev --background        # bun
```

Manage with: `astro dev stop` · `astro dev status` · `astro dev logs`

> `.astro` files are excluded from Biome. Run both `<pm> run astro check` and `<pm> run check` before finishing.

## Code Style (Biome — no ESLint/Prettier)

- Single quotes, semicolons `asNeeded`, trailing commas
- JSX attributes use double quotes
- 80-char line width, 2-space indent, LF line endings
- `.astro` files are excluded from Biome — type-check with `<pm> run astro check`

**Always run `<pm> run check` before finishing any code change.**

## Tailwind v4 Rules

### Customization

Add tokens to `src/styles/global.css` — never create a `tailwind.config.js`:

```css
@import "tailwindcss";

@theme {
  --color-brand: #6366f1;
  --default-font-family: "YourFont", sans-serif;  /* body font */
  --heading-font-family: "YourFont Display", sans-serif; /* headings */
}
```

**Font tokens**: Use `--default-font-family` (body) and `--heading-font-family` (headings, applied to `h1–h6` in `@layer base`). Load webfonts via `<link>` in `src/layouts/Layout.astro`'s `<head>`, not via `@import` in CSS.

**Never write these classes:**
- `font-[var(--heading-font-family)]`
- `font-(--heading-font-family)`
- `font-[var(--default-font-family)]`
- `font-(--default-font-family)`

They are redundant on heading elements and `<body>`. If you need the heading font on a non-heading element (e.g. a logo `<a>` or `<p>`), add the selector to `@layer base` or define a dedicated utility in `@theme` instead.

### Breaking Changes from v3

- `space-x-*` / `space-y-*` → use `gap-*` with flex/grid
- `divide-*` → use borders on individual children
- `shadow-sm` → `shadow-xs`
- `bg-opacity-*` / `text-opacity-*` → use `bg-black/50` syntax
- `flex-shrink` / `flex-grow` → `shrink` / `grow`
- `overflow-ellipsis` → `text-ellipsis`

### Color Tokens

Never use raw Tailwind scale colors for project-specific UI. Define named tokens:

```css
@theme {
  --color-dark: #111111;
  --color-muted: #666666;
  --color-border: #e5e5e5;
}
```

Use `text-muted` instead of `text-neutral-400`.

### Spacing

Prefer the standard Tailwind scale (`gap-2`) over arbitrary values (`gap-[8px]`). Add to `@theme` if a non-standard value is used more than once.

### Shared Class Consolidation

When 3+ sibling elements share the same classes, consolidate onto the parent with `*:`:

```html
<ul class="*:hover:text-white">
  <li><a href="#">About</a></li>
  <li><a href="#">Works</a></li>
  <li><a href="#">Contact</a></li>
</ul>
```

## Icons

Use `@lucide/astro` for tree-shakable SVG icons:

```astro
---
import { Camera } from '@lucide/astro';
---

<Camera size={16} class="text-muted" />
```

| Prop | Default |
|---|---|
| `size` | `24` |
| `color` | `currentColor` |
| `stroke-width` | `2` |

## Adding Dependencies

`.npmrc` enforces `min-release-age=3`. Build script permissions are managed via `allowBuilds` in `pnpm-workspace.yaml` and `allowScripts` in `package.json`. If a package fails to install:

- Needs build scripts → add to `allowBuilds` in `pnpm-workspace.yaml`
- Just published → add to `minimumReleaseAgeExclude` in `pnpm-workspace.yaml`

## Accessibility

Do not write uppercase text directly in HTML. Apply `text-transform: uppercase` via the `uppercase` Tailwind class instead. This prevents screen readers from spelling out each letter.

```html
<!-- ❌ -->
<a href="#concept">CONCEPT</a>

<!-- ✅ -->
<a href="#concept" class="uppercase">Concept</a>
```
