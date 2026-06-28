---
name: figma-workflow
description: >-
  Accepts a Figma URL and runs each phase — Implementation → Review → Optimization → Tailwind Review — as sequential sub-agents. Auto-detects project type (Astro / React / Vanilla) and branches accordingly.
argument-hint: "<figma-url>"
allowed-tools: Agent
---

# Figma → Code Workflow (Auto-detected Environment)

Accepts a Figma URL and runs **Implementation → Review → Optimization → Tailwind Review** as sequential sub-agents. Auto-detects the project type and branches accordingly.

## Arguments

The Figma URL is passed via `$ARGUMENTS`.

## Note: Large Pages

For tall pages like landing pages, passing a single large frame can bloat the context. In that case, split the design into **individual section frames** (Hero, Features, CTA, etc.) and run this workflow multiple times.

---

## Step 0: Environment Detection (run first)

Run the following Node.js script to detect the project type and store it as `ENV` for all subsequent phases:

```bash
node -e "const fs=require('fs');const astro=['astro.config.ts','astro.config.mjs','astro.config.js','astro.config.cjs'].some(f=>fs.existsSync(f));if(astro){console.log('astro')}else if(fs.existsSync('package.json')){const p=JSON.parse(fs.readFileSync('package.json','utf8'));const d={...p.dependencies,...p.devDependencies};(d['react']||d['react-dom'])?console.log('react'):console.log('vanilla')}else{console.log('vanilla')}"
```

| ENV | Phases | Flow |
|-----|--------|------|
| `astro` | 4 | Implementation → Review → Optimization → Tailwind Review |
| `react` | 4 | Implementation → Review → Optimization → Tailwind Review |
| `vanilla` | 3 | Implementation → Review → Tailwind Review |

---

## Execution Rules

- Launch each phase as a sub-agent using the **Agent tool**
- Always run **Phase 1 → Phase 2 → Phase 3 (→ Phase 4)** in order (no parallel execution)
- Confirm each agent has completed before launching the next
- After all phases are complete, summarize each agent's report for the user
- Replace `<pm>` in the prompts below with the project's package manager (`npm` / `yarn` / `pnpm`)

---

## Phase 1: Implementation

Launch a sub-agent using the appropriate prompt for the ENV.

### Astro Prompt

```
Implement the design from the following Figma URL: $ARGUMENTS

## Steps

### 0. Confirm Target File

If `$ARGUMENTS` does not include a filename (e.g. `about.astro`), confirm with the user before starting implementation:

> Please provide the target filename (e.g. `src/pages/index.astro`, `src/components/Hero.astro`)

- Also confirm whether this is a page (`src/pages/`) or a component (`src/components/`)
- Also confirm whether to add to an existing file or create a new one

### 1. Fetch Design Information
- Use `get_design_context` to fetch design context and reference code
- Use `get_variable_defs` to fetch design tokens (colors, spacing, etc.)
- Check any annotations in the file for implementation requirements

### 2. Review Existing Code
- Review existing components in `src/components/` and check if any can be reused
- Check already-defined tokens in the `@theme` block in `src/styles/global.css`

### 3. Implementation
Follow the CLAUDE.md guidelines strictly:

**Astro Component Structure**
- Place new components in `src/components/` (PascalCase filenames, e.g. `HeroSection.astro`)
- Place new pages in `src/pages/`
- In `.astro` files, write imports and logic in the frontmatter (`---`) section, and HTML in the template

**Styling**
- Use Tailwind CSS v4 syntax (use `gap-*` + flex/grid instead of `space-x/y-*`)
- Write classes directly in the `class` attribute (do not use JSX `className`)
- Add global styles to `src/styles/global.css`

**Accessibility**
- Use semantic HTML tags (`header`, `main`, `section`, `nav`, etc.)
- Add appropriate ARIA attributes (`aria-label`, `aria-expanded`, etc.)
- Set meaningful `alt` text on images

**Design Tokens**
- If tokens from Figma are not yet in `@theme`, add them to `src/styles/global.css`
- Use arbitrary values (`w-[42px]`) as a last resort; if used in multiple places, add to `@theme`

**Icons**
- Import and use icons from `@lucide/astro` as Astro components
  ```astro
  ---
  import { Camera } from '@lucide/astro';
  ---
  <Camera size={16} stroke-width={1} class="text-muted" />
  ```
- Props: `size` (default `24`), `color` (default `currentColor`), `stroke-width` (default `2`)

**Image Assets**
- Figma MCP image URLs (`https://www.figma.com/api/mcp/asset/...`) **expire after 7 days** — always save them locally
- Download to `src/assets/images/` in `webp` format, using kebab-case names that describe the content
  ```bash
  curl -s -o src/assets/images/<name>.webp "<figma-asset-url>"
  ```
- Reference via the `<Image />` component from `astro:assets` (never embed URLs directly):
  ```astro
  ---
  import { Image } from 'astro:assets';
  import heroImage from '../assets/images/hero.webp';
  ---
  <Image src={heroImage} alt="..." />
  ```

### 4. Completion Check
After implementation, confirm the following and include in your message:
- List of files created or modified
- Tokens newly added to `@theme` (if any)
- Status of existing component reuse
- List of downloaded image files (saved in `src/assets/images/`)
- Whether all requirements specified in annotations are met
```

---

### React Prompt

```
Implement the design from the following Figma URL: $ARGUMENTS

## Steps

### 0. Confirm Target File

If `$ARGUMENTS` does not include a filename, confirm with the user before starting implementation:

> Please provide the target filename (e.g. `src/pages/About.tsx`, `src/components/Hero.tsx`)

- Also confirm whether to add to an existing file or create a new one

### 1. Fetch Design Information
- Use `get_design_context` to fetch design context and reference code
- Use `get_variable_defs` to fetch design tokens (colors, spacing, etc.)
- Check any annotations in the file for implementation requirements

### 2. Review Existing Code
- Review existing components in `src/components/` and check if any can be reused
- Check already-defined tokens in the `@theme` block in `src/index.css`
- Always use the `cn()` function from `src/lib/utils.ts`

### 3. Implementation
Follow the CLAUDE.md guidelines strictly:

**Component Placement**
- Place new components in `src/components/`
- Use PascalCase filenames (e.g. `HeroSection.tsx`)

**Styling**
- Use Tailwind CSS v4 syntax (use `gap-*` + flex/grid instead of `space-x/y-*`)
- Always use `cn()` for class composition (`import { cn } from '@/lib/utils'`)
- Consider CVA (`class-variance-authority`) for components with multiple variants
- Consider `tailwind-variants` for styles spanning multiple child elements
- Use `lucide-react` for icons (inline: `w-4 h-4`, headings: `w-6 h-6`)

**TypeScript**
- Do not write `import React from 'react'` (using react-jsx transform)
- Use named imports for hooks: `import { useState } from 'react'`
- Always explicitly define props with TypeScript interfaces
- Extend existing HTML elements using `ComponentProps<'button'>` etc.

**Accessibility**
- Use semantic HTML tags (`header`, `main`, `section`, `nav`, etc.)
- Add appropriate ARIA attributes (`aria-label`, `aria-expanded`, `aria-controls`, etc.)
- Set meaningful `alt` text on images

**Design Tokens**
- If tokens from Figma are not yet in `@theme`, add them to `src/index.css`
- Use arbitrary values (`w-[42px]`) as a last resort; if used in multiple places, add to `@theme`

**Image Assets**
- Figma MCP image URLs (`https://www.figma.com/api/mcp/asset/...`) **expire after 7 days** — always save them locally
- Download to `src/assets/images/` in `webp` format, using kebab-case names that describe the content
  ```bash
  curl -s -o src/assets/images/<name>.webp "<figma-asset-url>"
  ```
- Reference via import (never embed URLs directly):
  ```tsx
  import heroImage from '@/assets/images/hero.webp'
  <img src={heroImage} alt="..." />
  ```

### 4. Completion Check
After implementation, confirm the following and include in your message:
- List of files created or modified
- Tokens newly added to `@theme` (if any)
- Status of existing component reuse
- List of downloaded image files (saved in `src/assets/images/`)
- Whether all requirements specified in annotations are met
```

---

### Vanilla Prompt

```
Implement the design from the following Figma URL: $ARGUMENTS

## Steps

### 0. Confirm Target File

If `$ARGUMENTS` does not include a filename (e.g. `about.html`), confirm with the user before starting implementation:

> Please provide the target HTML filename (e.g. `index.html`, `about.html`)

- Also confirm whether to implement in an existing file or create a new one
- For a new file, create it using the template from CLAUDE.md's "HTML Page Requirements"

### 1. Fetch Design Information
- Use `get_design_context` to fetch design context and reference code
- Use `get_variable_defs` to fetch design tokens (colors, spacing, etc.)
- Check any annotations in the file for implementation requirements

### 2. Review Existing Code
- Check already-defined tokens in the `@theme` block in `src/style.css`
- Check the Lucide icons already registered in `src/main.ts`

### 3. Implementation
Follow the CLAUDE.md guidelines strictly:

**HTML Structure**
- Write markup directly inside `<body>` of the target page
- Use semantic HTML tags (`header`, `main`, `footer`, `section`, `nav`, etc.)
- Set meaningful `alt` text on images

**Styling**
- Use Tailwind CSS v4 syntax (use `gap-*` + flex/grid instead of `space-x/y-*`)
- Write classes directly in the HTML `class` attribute
- Do not use CSS modules or external CSS files

**Design Tokens**
- If tokens from Figma are not yet in `@theme`, add them to `src/style.css`
- Use arbitrary values (`w-[42px]`) as a last resort; if used in multiple places, add to `@theme`

**Icons**
- Use Lucide for icons
- Import the icon into `src/main.ts` and register it with `createIcons()`:
  ```typescript
  import { createIcons, IconName } from 'lucide'
  createIcons({ icons: { IconName } })
  ```
- Use kebab-case in HTML: `<i data-lucide="icon-name"></i>`

**Image Assets**
- Figma MCP image URLs (`https://www.figma.com/api/mcp/asset/...`) **expire after 7 days** — always save them locally
- Download to `public/images/` in `webp` format, using kebab-case names that describe the content
  ```bash
  curl -s -o public/images/<name>.webp "<figma-asset-url>"
  ```
- Reference via root-relative path (never embed URLs directly):
  ```html
  <img src="/images/hero.webp" alt="..." />
  ```

### 4. Completion Check
After implementation, confirm the following and include in your message:
- List of files created or modified
- Tokens newly added to `@theme` (if any)
- Lucide icons added to `src/main.ts` (if any)
- List of downloaded image files (saved in `public/images/`)
- Whether all requirements specified in annotations are met
```

---

## Phase 2: Review and Fix

Launch a sub-agent using the appropriate prompt for the ENV.

### Astro Prompt

```
Review and compare the design from the following Figma URL with the implementation: $ARGUMENTS

## Steps

### 1. Fetch Figma Screenshot
- Use `get_screenshot` to take a screenshot of the target node
- Use the screenshot to identify visual differences from the implementation

### 2. Identify and Fix Visual Differences
Compare the screenshot with the implementation and check/fix the following:

**Layout**
- Element positioning and alignment (flex/grid direction, justify/align settings)
- Spacing (padding, margin, gap values)
- Sizes (width, height)

**Styles**
- Colors (`@theme` tokens or Tailwind color palette)
- Typography (font size, weight, line height)
- Border radius, box shadow
- Borders (color, width, style)

**Responsive**
- Appearance at mobile (`sm:`), tablet (`md:`), and desktop (`lg:`)

### 3. Resolve Type Check Errors
Run:
```bash
<pm> run astro check
```
and resolve all type errors in `.astro` files.

### 4. Code Style Check
Run:
```bash
<pm> run check
```
to auto-fix Biome lint and format errors (`.astro` files are excluded from Biome).

### 5. Accessibility Check
- Does keyboard navigation work?
- Are ARIA attributes (`aria-label`, etc.) set appropriately?
- Does color contrast meet WCAG AA (4.5:1)?
- Do images have meaningful `alt` text?

### 6. Review Report
After all fixes, summarize:
- Visual differences that were fixed
- Type error and code style fixes (if any)
- Accessibility improvements (if any)
- Remaining issues (if any)
```

---

### React Prompt

```
Review and compare the design from the following Figma URL with the implementation: $ARGUMENTS

## Steps

### 1. Fetch Figma Screenshot
- Use `get_screenshot` to take a screenshot of the target node
- Use the screenshot to identify visual differences from the implementation

### 2. Identify and Fix Visual Differences
Compare the screenshot with the implementation and check/fix the following:

**Layout**
- Element positioning and alignment (flex/grid direction, justify/align settings)
- Spacing (padding, margin, gap values)
- Sizes (width, height)

**Styles**
- Colors (`@theme` tokens or Tailwind color palette)
- Typography (font size, weight, line height)
- Border radius, box shadow
- Borders (color, width, style)

**Responsive**
- Appearance at mobile (`sm:`), tablet (`md:`), and desktop (`lg:`)

### 3. Resolve TypeScript Errors and Warnings
Run:
```bash
<pm> run build
```
and resolve all TypeScript errors and warnings.

Common issues:
- `noUnusedVariables`: remove unused variables and imports
- `noExplicitAny`: replace `any` with specific types
- `useExhaustiveDependencies`: fix dependency arrays in `useEffect` etc.

### 4. Code Style Check
Run:
```bash
<pm> run check
```
to auto-fix Biome lint and format errors.

### 5. Accessibility Check
- Does keyboard navigation work?
- Are ARIA attributes (`aria-label`, `aria-expanded`, `aria-controls`, etc.) set appropriately?
- Does color contrast meet WCAG AA (4.5:1)?
- Do images have meaningful `alt` text?

### 6. Review Report
After all fixes, summarize:
- Visual differences that were fixed
- TypeScript error and warning fixes (if any)
- Accessibility improvements (if any)
- Remaining issues (if any)
```

---

### Vanilla Prompt

```
Review and compare the design from the following Figma URL with the implementation: $ARGUMENTS

## Steps

### 1. Fetch Figma Screenshot
- Use `get_screenshot` to take a screenshot of the target node
- Use the screenshot to identify visual differences from the implementation

### 2. Identify and Fix Visual Differences
Compare the screenshot with the implementation and check/fix the following:

**Layout**
- Element positioning and alignment (flex/grid direction, justify/align settings)
- Spacing (padding, margin, gap values)
- Sizes (width, height)

**Styles**
- Colors (`@theme` tokens or Tailwind color palette)
- Typography (font size, weight, line height)
- Border radius, box shadow
- Borders (color, width, style)

**Responsive**
- Appearance at mobile (`sm:`), tablet (`md:`), and desktop (`lg:`)

### 3. Resolve TypeScript Errors
Run:
```bash
<pm> run build
```
and resolve all TypeScript errors.

Common issues:
- `noUnusedLocals` / `noUnusedParameters`: remove unused variables and imports

### 4. Accessibility Check
- Are semantic HTML tags used (`header`, `main`, `nav`, etc.)?
- Do images have meaningful `alt` text?
- Does color contrast meet WCAG AA (4.5:1)?

### 5. Review Report
After all fixes, summarize:
- Visual differences that were fixed
- TypeScript error fixes (if any)
- Accessibility improvements (if any)
- Remaining issues (if any)
```

---

## Phase 3: Optimization (Astro / React only)

Skip this phase if `ENV = vanilla` and proceed to Tailwind Review.

### Astro Prompt

```
Refactor the implemented components in `src/components/` to improve reusability and maintainability.

## Steps

### 1. Assess Current State
Review all files in `src/components/` and `src/pages/` and identify:
- Duplicated markup or styles
- Hardcoded values (colors, sizes, etc.)
- Groupings that could be extracted as components

### 2. Organize Components
- Consider componentizing markup that appears in 2 or more places
- Extract components of appropriate granularity from pages into `src/components/`
- Use Astro's `<slot />` to design flexible components
- Avoid over-abstraction (clarity over DRY)

### 3. Organize Design Tokens
- Move hardcoded color and spacing values to the `@theme` block in `src/styles/global.css`
- Tokenize arbitrary values (`w-[42px]`) used in multiple places
- Use semantic token names (e.g. `--color-brand-primary`)

### 4. Final Code Style Check
Run:
```bash
<pm> run astro check && <pm> run check
```
and confirm there are no errors.

### 5. Optimization Report
Summarize:
- Files refactored and what changed
- Tokens added or reorganized in `@theme`
- New components created (if any)
```

---

### React Prompt

```
Refactor the implemented components in `src/components/` to improve reusability and maintainability.

## Steps

### 1. Assess Current State
Review all files in `src/components/` and identify:
- Duplicated styles or logic
- Hardcoded values (colors, sizes, etc.)
- Opportunities to migrate to better component patterns

### 2. Optimize Component Patterns
Per the "Component Patterns" section of CLAUDE.md, choose the best approach for each component:

**cn function** — simple components with few conditional classes
**CVA (class-variance-authority)** — single-element components with multiple variants (buttons, badges, etc.)
**tailwind-variants** — components with variants spanning multiple child elements (cards, forms, etc.)

### 3. Organize Design Tokens
- Move hardcoded color and spacing values to the `@theme` block in `src/index.css`
- Tokenize arbitrary values (`w-[42px]`) used in multiple places
- Use semantic token names (e.g. `--color-brand-primary`)

### 4. Extract Common Patterns
- Consider creating a shared component when the same style combination is used in 3+ places
- Avoid over-abstraction (clarity over DRY)

### 5. Strengthen Type Safety
- Eliminate all `any` types
- Use `ComponentProps<'element'>` to inherit HTML attributes
- Accept a `className` prop and merge with `cn()` to allow external overrides

### 6. Final Code Style Check
Run:
```bash
<pm> run check
```
to confirm compliance with Biome rules.

### 7. Optimization Report
Summarize:
- Components refactored and what changed
- Tokens added or reorganized in `@theme`
- New shared components created (if any)
- Component patterns selected and why
```

---

## Phase 4 (Astro / React) / Phase 3 (Vanilla): Tailwind CSS Review

Common to all environments. Launch a sub-agent via the Agent tool with the following prompt:

```
Review and optimize the Tailwind CSS code in the project.

Load `.claude/skills/tailwind-review/SKILL.md` directly with the Read tool and follow its instructions.

Apply all fixes automatically without confirmation (running inside a pipeline).

> **Note:** If the Skill tool cannot invoke `tailwind-review` (e.g. CWD mismatch), resolve the path with `node -e "const path=require('path'),os=require('os');console.log(path.join(os.homedir(),'.claude','skills','tailwind-review','SKILL.md'))"` and read it directly with the Read tool.
```

---

## Final Report

After all phases complete, report in the format appropriate for the ENV.

**Astro / React (4 phases)**

```
## Figma Workflow Complete

### Phase 1: Implementation
[Summary of Agent 1's report]

### Phase 2: Review
[Summary of Agent 2's report]

### Phase 3: Optimization
[Summary of Agent 3's report]

### Phase 4: Tailwind CSS Review
[Summary of Agent 4's report]
```

**Vanilla (3 phases)**

```
## Figma Workflow Complete

### Phase 1: Implementation
[Summary of Agent 1's report]

### Phase 2: Review
[Summary of Agent 2's report]

### Phase 3: Tailwind CSS Review
[Summary of Agent 3's report]
```
