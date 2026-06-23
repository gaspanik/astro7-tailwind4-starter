---
paths:
  - "src/**/*.{astro,css}"
---

# Tailwind CSS v4 Rules

Tailwind v4 runs via the `@tailwindcss/vite` plugin. Config is CSS-based — never create `tailwind.config.js`. Global styles live in `src/styles/global.css` (`@import "tailwindcss"` plus `@theme` tokens).

## v4 syntax only

**Do not use v3 class names.** Common mistakes:

- `space-x-*` / `space-y-*` → use `gap-*` with flex/grid
- `divide-*` → use borders on individual children
- `shadow-sm` → `shadow-xs`
- `bg-opacity-*` / `text-opacity-*` → use `bg-black/50` syntax
- `flex-shrink` / `flex-grow` → `shrink` / `grow`
- `overflow-ellipsis` → `text-ellipsis`

## Values and tokens

- Use the standard Tailwind scale (1 unit = 4px) over arbitrary values: `gap-2` beats `gap-[8px]`. Add a `@theme` token if a non-standard value is used more than once
- Avoid raw Tailwind scale colors (`gray-*`, `neutral-*`, …) for the project palette. Define all project colors as `@theme` tokens, then use `text-muted` instead of `text-neutral-400`:

```css
/* src/styles/global.css */
@theme {
  --color-dark: #111111;
  --color-muted: #666666;
  --color-border: #e5e5e5;
}
```

## Font family

`--default-font-family` is the body default (set in `@theme`) and `--heading-font-family` is already applied to h1–h6 via `@layer base` — both in `src/styles/global.css`. Never write these redundant classes:

- `font-[var(--heading-font-family)]` / `font-(--heading-font-family)`
- `font-[var(--default-font-family)]` / `font-(--default-font-family)`

To use the heading font on a non-heading element (e.g. a logo `<a>`), extend `@layer base` or define a dedicated utility in `@theme` instead.

## Shared class consolidation (`*:`)

When 3+ sibling elements share repeated classes, consolidate onto the parent with the `*:` variant (applies to direct children only):

```html
<ul class="*:hover:text-white">
  <li><a href="#">About</a></li>
  <li><a href="#">Works</a></li>
  <li><a href="#">Contact</a></li>
</ul>
```

## Uppercase text

Never write text in ALL CAPS directly in markup — write proper case and apply the `uppercase` class. Screen readers may spell out capitalized text letter by letter; brand names and proper nouns are exempt.

```html
<!-- ❌ <a href="#about">ABOUT</a> -->
<!-- ✅ <a href="#about" class="uppercase">About</a> -->
```
