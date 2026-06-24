# Astro 7 + Tailwind CSS v4 Starter

A minimal, opinionated starter for building static sites and web apps with **Astro 7**, **Tailwind CSS v4**, **Lucide icons**, and **Biome**.

## Stack

| Tool | Version | Role |
|---|---|---|
| [Astro](https://astro.build) | ^7 | Framework / SSG |
| [Tailwind CSS](https://tailwindcss.com) | ^4 | Styling (via Vite plugin) |
| [@lucide/astro](https://lucide.dev) | ^1 | Tree-shakable SVG icons |
| [Biome](https://biomejs.dev) | 2.x (pinned) | Lint + Format |
| TypeScript | ^6 | Type safety (strict mode) |

## Project Structure

```text
/
├── .github/
│   ├── copilot-instructions.md  # GitHub Copilot guidelines
│   └── prompts/                 # Reusable prompt templates (e.g. new-page)
├── .vscode/
│   ├── extensions.json          # Recommended extensions
│   └── launch.json              # Debug configuration
├── public/
│   ├── favicon.ico
│   └── favicon.svg
├── src/
│   ├── assets/          # Vite-processed assets (import to reference)
│   ├── components/      # Reusable .astro components
│   ├── layouts/
│   │   └── Layout.astro # Root HTML shell — global CSS imported here
│   ├── pages/           # File-based routes (each .astro → a URL)
│   └── styles/
│       └── global.css   # @import "tailwindcss" + @theme tokens
├── AGENTS.md            # AI agent guidelines (OpenAI Codex / generic)
├── CLAUDE.md            # Claude Code guidelines
├── GEMINI.md            # Gemini CLI guidelines
├── astro.config.mjs
├── biome.json
└── package.json
```

## Commands

Replace `<pm>` with your package manager (`npm`, `yarn`, `pnpm`, etc.).

```sh
<pm> install           # Install dependencies
<pm> run dev           # Start dev server at localhost:4321
<pm> run build         # Build production site to ./dist/
<pm> run preview       # Preview production build locally
<pm> run astro check   # Type-check .astro files
<pm> run lint          # Biome lint --write
<pm> run format        # Biome format --write
<pm> run check         # Biome check --write (lint + format combined)
<pm> run astro-upgrade # Upgrade Astro via @astrojs/upgrade
```

## Tailwind v4 Configuration

No `tailwind.config.js`. All theme customization lives in `src/styles/global.css`:

```css
@import "tailwindcss";

@theme {
  --color-brand: #6366f1;
  --font-sans: "Inter", sans-serif;
}
```

Define project-specific colors as `@theme` tokens (e.g. `--color-muted`) instead of using raw Tailwind scale utilities.

## Icons

Import any [Lucide](https://lucide.dev/icons/) icon by name from `@lucide/astro`:

```astro
---
import { Camera } from '@lucide/astro'
---

<Camera size={24} class="text-muted" />
```

## Code Style

No ESLint or Prettier. JS/TS/JSON/CSS is managed by **Biome**. `.astro` files are excluded from Biome — use `<pm> run astro check` for type-checking them.

- Single quotes, semicolons `asNeeded`, trailing commas; JSX attributes use double quotes
- 80-char line width, 2-space indent, LF line endings (`.editorconfig`)
- Always run both `<pm> run astro check` and `<pm> run check` before finishing any code change

## Security

`.npmrc` sets `min-release-age=3`, blocking packages published fewer than 3 days ago.

Build script permissions are managed via `allowBuilds` in `pnpm-workspace.yaml` and `allowScripts` in `package.json`. Only explicitly listed packages (e.g. `esbuild`, `sharp`, `fsevents`) may run install scripts.

## Astro v7 Notable Changes

If upgrading from v6, be aware of these breaking changes:

- **Strict HTML validation** — The Rust compiler no longer auto-corrects invalid HTML. Unclosed tags produce errors.
- **Whitespace handling** — Default changed to JSX-style (`compressHTML: 'jsx'`): whitespace between inline elements is collapsed. Add explicit spaces where needed, or set `compressHTML: true` in `astro.config.mjs` to restore the previous behavior.
- **`src/fetch.ts` is reserved** — Used by Astro's advanced routing. Rename any file at that path and set `fetchFile` in config.
- **Vite 8** — `package.json` includes an `overrides` entry pinning Vite to `^8`.
- **Sätteri markdown** — New default Markdown processor (Rust-based). To keep remark/rehype plugins, reinstall `@astrojs/markdown-remark` and configure it explicitly.
- **Background dev server** — `astro dev --background` starts the server detached from the terminal; manage with `astro dev stop` / `astro dev status` / `astro dev logs`.
