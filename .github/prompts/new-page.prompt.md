---
name: new-page
description: Scaffold a new Astro page with layout, route, and optional components
---

Create a new Astro page for this project.

Page name or route: {{input}}

Requirements:
- File goes in `src/pages/` — the filename becomes the URL route
- Wrap content in the `Layout` component from `src/layouts/Layout.astro`
- Use Tailwind utility classes (v4 syntax) — no inline styles
- Define any new color/spacing tokens in `src/styles/global.css` under `@theme`
- If the page needs reusable UI blocks, create matching components in `src/components/`
- Use `@lucide/astro` for any icons
- Do NOT write uppercase text in HTML — apply the `uppercase` Tailwind class instead
- Run `pnpm astro check && pnpm check` after creating all files
