---
name: tailwind-typescale
description: >
  Apply a harmonious type scale to a Tailwind CSS v4 project by choosing a base
  font size and a named scale ratio (from typescale.com). Sets --text-xs through
  --text-9xl in @theme (covering all Tailwind text size classes) and maps h1–h6
  font sizes in @layer base.
  Use this skill when the user says things like:
  "タイプスケール変えたいんだけど",
  "フォントサイズを整えたい",
  "見出しのサイズをちゃんとしたい",
  "タイポグラフィのスケールを設定して",
  "apply a type scale", "set up typography scale", "change font sizes".
argument-hint: "[base-size] [scale-name]"
allowed-tools: Bash, Read, Edit, AskUserQuestion
---

# Apply Type Scale

Sets up a harmonious typography scale for Tailwind CSS v4 projects.

---

## Step 0: Pre-flight Checks

Run all checks before asking the user anything.

1. Search for CSS files containing `@import "tailwindcss"` or `@import 'tailwindcss'` across the project (excluding `node_modules`/`dist`)
2. Check whether `DESIGN.md` exists in the project root
3. Check whether `tailwind.config.js` exists in the project root (v3 indicator)

### Step 0-A: Determine the target CSS file

**If exactly 1 file found:** Use it as the target. Proceed to Step 0-B.

**If multiple files found:** Ask via `AskUserQuestion` which file to apply the scale to (list each path as an option). Use the chosen file as the target.

**If no file found:** Run a fallback search for `@theme` blocks:

Search for CSS files containing an `@theme` block across the project (excluding `node_modules`/`dist`).

Apply the same 1 / multiple / none logic above.

**If still no file found:** Ask the user to provide the path to their Tailwind CSS entry file (the one containing `@import "tailwindcss"`), then stop with instructions.

### Step 0-B: Report findings

Check for existing `--text-*` variables in the target file:

Check whether the target file contains any `--text-*` CSS variables.

Display a summary of what was found:

```
## Current project state

Target CSS:         {path/to/file.css}
DESIGN.md:          found / not found
tailwind.config.js: found (v3) / not found (v4)

Type scale (--text-*):
  Found:
    --text-xs:   0.600rem  (existing)
    --text-sm:   0.750rem  (existing)
    ... (list all found variables)
  Not found
```

**If `DESIGN.md` exists:**
Read the file and check for any typography-related content (font-size, type scale, scale name, etc.).
If found, display: "DESIGN.md contains typography settings:" followed by the relevant excerpt.

**If `tailwind.config.js` exists (v3):**
This skill requires Tailwind CSS v4. Warn and stop:

```
⚠️  tailwind.config.js detected — this project appears to use Tailwind CSS v3.
This skill requires v4 (@theme directive and --text-* variables).
For v3 projects, configure font sizes under theme.fontSize in tailwind.config.js.
```

### Step 0-C: Confirm overwrite if --text-* variables already exist

If one or more `--text-*` variables are found, ask via `AskUserQuestion`:

- Overwrite with a new scale
- Cancel (keep existing)

If the user cancels, stop.
If the user confirms, proceed to Step 1.

If no `--text-*` variables exist, proceed directly to Step 1.

---

## Step 1: Gather Parameters

Interpret `$ARGUMENTS` as follows:

- No arguments → Step 1-A (ask via AskUserQuestion)
- Arguments provided → parse as `[base-size] [scale-name]` space-separated.
  - e.g. `16px perfect-fourth`, `18 golden-ratio`
  - Ask for any missing values in Step 1-A.

### Step 1-A: Ask via AskUserQuestion

Present both questions **at the same time** in a single `AskUserQuestion` call.

**Q1: Base font size**
- 16px – Standard (recommended)
- 14px – Compact
- 18px – Spacious
- 20px – Large

**Q2: Type scale**
- Perfect Fourth 1.333 – Balanced and readable (recommended)
- Major Third 1.250 – Gentle, compact
- Golden Ratio 1.618 – Dynamic contrast
- Minor Third 1.200 – Subtle, refined

After both answers, proceed to Step 2.

---

## Step 2: Calculate the Scale

Use the following formula:

```
base_rem = base font size (px) / 16
ratio    = chosen scale ratio

text-xs   = base_rem / ratio^2
text-sm   = base_rem / ratio^1
text-base = base_rem
text-lg   = base_rem * ratio^1
text-xl   = base_rem * ratio^2
text-2xl  = base_rem * ratio^3
text-3xl  = base_rem * ratio^4
text-4xl  = base_rem * ratio^5
text-5xl  = base_rem * ratio^6
text-6xl  = base_rem * ratio^7
text-7xl  = base_rem * ratio^8
text-8xl  = base_rem * ratio^9
text-9xl  = base_rem * ratio^10
```

Round each value to **3 decimal places** (e.g. `1.333rem`).

### Recommended line-height per step (for `leading-*` utilities)

> **Important:** Do NOT embed line-height in `--text-*` variables. In Tailwind v4, `--text-base: 1rem 1.5` generates `font-size: 1rem 1.5` which is invalid CSS — the browser ignores it and falls back to 16px. Always use only the rem value: `--text-base: 1rem`.

Use Tailwind's `leading-*` utilities to control line-height per element:

| Step | Suggested leading-* |
|---|---|
| text-xs / text-sm | `leading-tight` (1.25) |
| text-base / text-lg | `leading-normal` (1.5) |
| text-xl / text-2xl | `leading-snug` (1.375) |
| text-3xl | `leading-snug` (1.375) |
| text-4xl / text-5xl | `leading-tight` (1.25) |
| text-6xl and above | `leading-none` (overridden to 1.1 in @theme) |

> **Note:** h1–h6 elements get `line-height: var(--line-height-none)` applied directly in `@layer base` (Step 4), using the `--line-height-none: 1.1` override written to `@theme`. Do NOT add `leading-none` as a Tailwind utility class or as a CSS property — it is set via `line-height` in `@layer base`.

---

## Step 3: Show Preview and Confirm

Display the calculated values as text (do not use AskUserQuestion here):

```
## Type Scale Preview

Config: base {Xpx} × {Scale name} ({ratio})

| Tailwind class | Size (rem) | Size (px) |
|---|---|---|
| text-xs   | X.XXXrem | XX.Xpx |
| text-sm   | X.XXXrem | XX.Xpx |
| text-base | X.XXXrem | XX.Xpx |
| text-lg   | X.XXXrem | XX.Xpx |
| text-xl   | X.XXXrem | XX.Xpx |
| text-2xl  | X.XXXrem | XX.Xpx |
| text-3xl  | X.XXXrem | XX.Xpx |
| text-4xl  | X.XXXrem | XX.Xpx |
| text-5xl  | X.XXXrem | XX.Xpx |
| text-6xl  | X.XXXrem | XX.Xpx |
| text-7xl  | X.XXXrem | XX.Xpx |
| text-8xl  | X.XXXrem | XX.Xpx |
| text-9xl  | X.XXXrem | XX.Xpx |

h1–h6 mapping:
  h1 → text-5xl ({size}rem)
  h2 → text-4xl ({size}rem)
  h3 → text-3xl ({size}rem)
  h4 → text-2xl ({size}rem)
  h5 → text-xl  ({size}rem)
  h6 → text-lg  ({size}rem)
```

Then ask via `AskUserQuestion`:
- Apply this scale (recommended)
- Try a different scale
- Cancel

If "Try a different scale" → go back to Step 1.
If "Cancel" → stop.

---

## Step 3.5: Backup Check

Before writing any changes, run:

```bash
node -e "const {execSync}=require('child_process');try{const o=execSync('git status --short',{encoding:'utf8',stdio:['pipe','pipe','pipe']});console.log(o.trim()||'clean')}catch{console.log('not a git repo')}"
```

- **Git repo with uncommitted changes:** Warn the user that `{target file}` will be overwritten. Recommend running `git stash` or `git commit` first, then ask via `AskUserQuestion`:
  - Continue anyway
  - Cancel (I'll stash/commit first)

  If the user cancels, stop.

- **Git repo, clean working tree:** Proceed silently.
- **Not a git repo:** Warn that there is no version control, and recommend making a manual backup of `{target file}` before continuing. Ask via `AskUserQuestion`:
  - Continue anyway
  - Cancel (I'll back up the file first)

  If the user cancels, stop.

---

## Step 4: Edit the target CSS file

Add or overwrite `--text-*` variables in the `@theme` block of the detected target file.

### Rules

- Overwrite existing `--text-*` variables if present
- Append before the closing `}` of `@theme` if not present
- Create a new `@theme` block if none exists

### Example output

```css
  /* Type scale: {Scale name} ({ratio}) — base {Xpx} */
  --text-xs: X.XXXrem;
  --text-sm: X.XXXrem;
  --text-base: X.XXXrem;
  --text-lg: X.XXXrem;
  --text-xl: X.XXXrem;
  --text-2xl: X.XXXrem;
  --text-3xl: X.XXXrem;
  --text-4xl: X.XXXrem;
  --text-5xl: X.XXXrem;
  --text-6xl: X.XXXrem;
  --text-7xl: X.XXXrem;
  --text-8xl: X.XXXrem;
  --text-9xl: X.XXXrem;
  --line-height-none: 1.1;
```

The `--line-height-none` override sets `leading-none` to `1.1` instead of Tailwind's default `1.0`, giving headings a slightly looser line-height that works better at display sizes.

Include a comment recording the scale name and base size for future reference.

### h1–h6 font-size and line-height mapping

Add `font-size` and `line-height` declarations to the `h1`–`h6` block inside `@layer base`.

> **CRITICAL:** Write only valid CSS properties. `leading-none`, `leading-tight`, etc. are **Tailwind utility classes**, not CSS properties. Writing `leading-none;` inside a CSS rule is invalid and will be silently ignored by the browser. Always use `line-height: var(--line-height-none)` — never `leading-none;`.

If `h1`–`h6` already exists in `@layer base`, add `font-size` and `line-height` to each rule.
If not, add a new ruleset:

```css
@layer base {
  h1 { font-size: var(--text-5xl); line-height: var(--line-height-none); }
  h2 { font-size: var(--text-4xl); line-height: var(--line-height-none); }
  h3 { font-size: var(--text-3xl); line-height: var(--line-height-none); }
  h4 { font-size: var(--text-2xl); line-height: var(--line-height-none); }
  h5 { font-size: var(--text-xl);  line-height: var(--line-height-none); }
  h6 { font-size: var(--text-lg);  line-height: var(--line-height-none); }
}
```

If an existing `h1`–`h6` block already has properties like `font-family`, preserve them and only add `font-size` and `line-height`.

---

## Step 4.5: Clean up explicit text-size and leading classes from h1–h6

After applying the scale, scan HTML files for heading elements that have explicit `text-*` size classes or `leading-*` classes.
- `text-*` size classes override the `@layer base` mapping and prevent the new scale from taking effect.
- `leading-*` classes override the `line-height: var(--line-height-none)` set in `@layer base` and must be removed.

> **ALWAYS run the searches below — do not skip, even if the project was just created or cloned from a template.** Template starters commonly include demo HTML files (e.g. `index.html`, `about.html`) that contain heading elements with explicit `text-*` size classes. Skipping this step because the project "looks new" will leave those classes in place and silently break the new scale.

> **Note:** Do NOT replace `leading-*` with `leading-none`. Line-height is now handled entirely by `@layer base` — just remove the class and let CSS take over.

### Detection

Run both searches:

```bash
node -e "const fs=require('fs'),path=require('path');const EXTS=['.html','.tsx','.jsx','.vue','.astro'],SKIP=['node_modules','dist'];const RE=[/<h[1-6][^>]*class(?:Name)?=\"[^\"]*text-(?:xs|sm|base|lg|xl|2xl|3xl|4xl|5xl|6xl|7xl|8xl|9xl)/,/<h[1-6][^>]*class(?:Name)?=\"[^\"]*leading-/];function walk(d){return fs.readdirSync(d,{withFileTypes:true}).flatMap(e=>{const p=path.join(d,e.name);return SKIP.some(s=>p.includes(s))?[]:e.isDirectory()?walk(p):EXTS.some(x=>e.name.endsWith(x))?[p]:[]});}let hits=0;for(const f of walk('.')){const ls=fs.readFileSync(f,'utf8').split('\n');ls.forEach((l,i)=>{if(RE.some(r=>r.test(l))){console.log(f+':'+(i+1)+': '+l.trim());hits++;}});}if(!hits)console.log('(no matches)');"
```

> React components may also build heading classes dynamically (template literals, `cn()` etc.) — the grep only catches literal `class` / `className` strings. If the project is React-based, additionally check heading components visually.

### Candidate list

If any matches are found, display them as plain text before asking:

```
The following h1–h6 elements have classes that conflict with the new scale:

  [1] post.html:68   <h2 class="text-ink font-semibold text-xl leading-tight mt-10 mb-4">
                      → text-xl removed (h2 uses text-4xl via @layer base)
                      → leading-tight removed (line-height is set in @layer base)
  [2] index.html:22  <h2 class="mt-2 font-medium text-primary text-base">
                      → text-base removed (h2 uses text-4xl via @layer base)
```

Then ask via `AskUserQuestion`:
- Apply all changes (recommended)
- Choose individually
- Skip (keep all as-is)

If "Choose individually" → list each item as a numbered question and let the user type which numbers to apply.
If "Skip" → proceed to Step 4.6 without changes.

### Applying changes

For each matched heading element:
1. Remove only the `text-*` size class token from the `class` attribute (preserve all other classes).
2. Remove any `leading-*` class tokens from the `class` attribute.

> **Note:** Do not remove `text-*` color classes (e.g. `text-ink`, `text-muted`, `text-primary`).
> Only remove size utilities: `text-xs`, `text-sm`, `text-base`, `text-lg`, `text-xl`, `text-2xl` … `text-9xl`.
> Remove `leading-*` classes entirely — do NOT replace them with `leading-none` or any other value.

---

## Step 4.6: Update DESIGN.md typography section

If `DESIGN.md` exists in the project root, its typography section may now be out of sync with the new scale.

Read the current `DESIGN.md` and check whether the `typography:` block in the YAML frontmatter contains font-size values. If so, display:

```
DESIGN.md contains typography tokens that reference font sizes.
The new scale changes these values. Update DESIGN.md to match?
```

Ask via `AskUserQuestion`:
- Update DESIGN.md (recommended)
- Skip

If "Update DESIGN.md":
1. Recalculate px values for the relevant heading levels using the new scale (h1 → text-5xl, h2 → text-4xl, etc.)
2. Update `fontSize` values in the `typography:` YAML block to match
3. Update the `## Typography` markdown section body to reflect the new scale name and ratio

If no font-size values are found in DESIGN.md, skip silently.

---

## Step 5: Done

Detect the package manager from the lockfile before reporting:

Detect the package manager by checking for a lockfile (`pnpm-lock.yaml`, `yarn.lock`, `package-lock.json`, `bun.lockb`).

| Lockfile | Package manager |
|---|---|
| `pnpm-lock.yaml` | `pnpm` |
| `yarn.lock` | `yarn` |
| `package-lock.json` | `npm` |
| `bun.lockb` | `bun` |

Use the detected package manager in the completion message:

```
## Applied

{Scale name} scale (base {Xpx}, ratio {ratio}) applied to {target file}.

Changes:
  - @theme: --text-xs through --text-9xl set (rem values only)
  - @theme: --line-height-none overridden to 1.1
  - @layer base: h1–h6 font-size and line-height mapped to scale
  - HTML: explicit text-* removed from X heading elements  ← if any were removed
  - DESIGN.md: typography section updated                  ← if updated

Changes take effect immediately if the dev server is running.
To start: <pm> run dev

💡 Heading line-height is set to 1.1 via @layer base (line-height: var(--line-height-none)).
   To adjust globally, change --line-height-none in your @theme block:

     --line-height-none: 1.1;   /* try 1.0 for tighter, 1.2 for looser */

   To override individual headings, add a CSS rule in @layer base or use an inline style.
   Do NOT use leading-* Tailwind classes on headings — they override @layer base and break consistency.
```

---

## Notes

- This skill is for Tailwind CSS **v4** only (`@theme` directive and `--text-*` variables)
- For v3 projects, configure font sizes under `theme.fontSize` in `tailwind.config.js`
- Covers all Tailwind text size classes: `text-xs` through `text-9xl`
