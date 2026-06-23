# Astro 7 + Tailwind CSS v4 Starter: Project Context

## Design Reference

If `DESIGN.md` exists in the project root, read it before starting any UI or feature work. It contains the site's design intent, color palette, typography, layout guidelines, and component conventions — treat it as the source of truth for visual and UX decisions.

This project is a modern, performance-oriented starter for building static sites and web applications using Astro 7 and Tailwind CSS v4.

# MCP Tools

### Astro Docs MCP (`mcp__AstroDocs__search_astro_docs`)

Use this tool when working with Astro-specific APIs, integrations, or configuration. Prefer it over training-data knowledge for:

- Astro component syntax, frontmatter, slots, `<slot />` usage
- Integrations: `@astrojs/react`, `@astrojs/tailwind`, `@astrojs/image`, etc.
- Content Collections, Actions, View Transitions, SSR/SSG config
- `astro.config.mjs` options and Vite plugin setup
- Any Astro 7 feature that may differ from earlier versions

Always check the docs when the behavior is uncertain or the user hits an unexpected error.

## Project Overview

- **Framework:** Astro 7 (Static Site Generator / Web Framework)
- **Styling:** Tailwind CSS v4 (configured via Vite plugin, no standalone config file)
- **Icons:** `@lucide/astro` for tree-shakable SVG icons
- **Tooling:** Biome (v2.x) for linting and formatting (excluding `.astro` files)
- **Type Safety:** TypeScript ^6 in strict mode
- **Package Manager:** pnpm (Node >= 22.12.0)

## Architecture & Directory Structure

- `src/layouts/Layout.astro`: The root HTML shell. Global CSS is imported here.
- `src/pages/`: File-based routing. Each `.astro` file corresponds to a URL.
- `src/components/`: Reusable UI components.
- `src/styles/global.css`: Entry point for Tailwind CSS. Contains `@theme` token definitions and base styles.
- `src/assets/`: Assets processed by Vite.
- `public/`: Static assets served directly.

## Building and Running

| Command | Description |
| :--- | :--- |
| `pnpm install` | Install all project dependencies |
| `pnpm dev` | Start the development server at `localhost:4321` |
| `pnpm build` | Build the production-ready site to `./dist/` |
| `pnpm preview` | Locally preview the production build |
| `pnpm astro check` | Run type-checking on `.astro` files |
| `pnpm check` | Run Biome lint and format (with `--write`) |
| `pnpm lint` | Run Biome lint only |
| `pnpm format` | Run Biome format only |

## Development Conventions

### Styling (Tailwind v4)
- **No `tailwind.config.js`:** All customizations (colors, fonts, etc.) must be defined in `src/styles/global.css` using the `@theme` block.
- **Theme Tokens:** Prefer using `@theme` tokens (e.g., `--color-brand`) over raw Tailwind scale utilities for project-specific values.
- **v4 Class Changes:**
  - `space-x-*` / `space-y-*` → use `gap-*` with flex/grid.
  - `shadow-sm` → `shadow-xs`.
  - `flex-shrink` / `flex-grow` → `shrink` / `grow`.
  - Use `bg-black/50` syntax instead of `bg-opacity-*`.
- **Consolidation:** Use `*:` variants to apply classes to multiple siblings via the parent when applicable (e.g., `*:hover:text-white`).

### Font Family
`--heading-font-family` is already applied to h1–h6 via `@layer base` in `src/style.css`.
`--default-font-family` is set as the default body font via `@theme`.
**Never write these classes:**
- `font-[var(--heading-font-family)]`
- `font-(--heading-font-family)`
- `font-[var(--default-font-family)]`
- `font-(--default-font-family)`

They are redundant on heading elements and `<body>`.
If you need the heading font on a non-heading element (e.g. a logo `<a>` or `<p>`),
add the selector to `@layer base` or define a dedicated utility in `@theme` instead.

### Code Style & Linting
- **Biome:** Handles JS, TS, JSON, and CSS.
  - **Quotes:** Single quotes for JS/TS, double quotes for JSX/HTML attributes.
  - **Semicolons:** Set to `asNeeded`.
  - **Indentation:** 2 spaces.
- **Astro Files:** `.astro` files are excluded from Biome. Use `pnpm astro check` for validation.
- **Pre-commit:** Always run `pnpm check` before finishing any code change.

### Icons
- Import icons from `@lucide/astro` (e.g., `import { Camera } from '@lucide/astro'`).
- Props: `size` (default 24), `color` (default currentColor), `stroke-width` (default 2).
- Use the `class` prop for Tailwind styling: `<Camera class="text-muted" />`.

## Security & Supply Chain
- **Package Maturity:** `min-release-age=3` (days) is enforced via `.npmrc` to block newly published packages.
- **Build Scripts:** Permitted packages (e.g. `esbuild`, `sharp`, `fsevents`) are listed in `allowBuilds` in `pnpm-workspace.yaml` and `allowScripts` in `package.json`. Only these may run install scripts.
- **Dependency Management:** If a new dependency fails to install, add it to `allowBuilds` in `pnpm-workspace.yaml` (if it needs build scripts) or `minimumReleaseAgeExclude` (if it was just published).

