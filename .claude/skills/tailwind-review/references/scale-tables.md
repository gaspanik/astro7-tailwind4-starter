# Pixel / Unit → Tailwind Scale Conversion Tables

Reference for Dimension 3-B of tailwind-review. Read when arbitrary px / em / rem values are found.

Suggest the closest match. If the value falls between steps, mention both neighbours and ask the user to confirm.

---

**Font size (`text-[*]`)**

| Arbitrary | Tailwind utility | Actual size |
|---|---|---|
| `text-[10px]` | `text-xs` | 0.75rem / 12px |
| `text-[12px]` | `text-xs` | 0.75rem / 12px |
| `text-[14px]` | `text-sm` | 0.875rem / 14px |
| `text-[16px]` | `text-base` | 1rem / 16px |
| `text-[18px]` | `text-lg` | 1.125rem / 18px |
| `text-[20px]` | `text-xl` | 1.25rem / 20px |
| `text-[24px]` | `text-2xl` | 1.5rem / 24px |
| `text-[30px]` | `text-3xl` | 1.875rem / 30px |
| `text-[36px]` | `text-4xl` | 2.25rem / 36px |
| `text-[48px]` | `text-5xl` | 3rem / 48px |
| `text-[60px]` | `text-6xl` | 3.75rem / 60px |
| `text-[72px]` | `text-7xl` | 4.5rem / 72px |

**Line height (`leading-[*]`)**

| Arbitrary | Tailwind utility | Value |
|---|---|---|
| `leading-[1]` | `leading-none` | 1 |
| `leading-[1.25]` | `leading-tight` | 1.25 |
| `leading-[1.375]` | `leading-snug` | 1.375 |
| `leading-[1.5]` | `leading-normal` | 1.5 |
| `leading-[1.625]` | `leading-relaxed` | 1.625 |
| `leading-[2]` | `leading-loose` | 2 |

**Letter spacing (`tracking-[*]`)**

| Arbitrary | Tailwind utility | Value |
|---|---|---|
| `tracking-[-0.05em]` | `tracking-tighter` | -0.05em |
| `tracking-[-0.025em]` | `tracking-tight` | -0.025em |
| `tracking-[0em]` | `tracking-normal` | 0em |
| `tracking-[0.025em]` | `tracking-wide` | 0.025em |
| `tracking-[0.05em]` | `tracking-wider` | 0.05em |
| `tracking-[0.1em]` | `tracking-widest` | 0.1em |

**Font weight (`font-[*]`)**

| Arbitrary | Tailwind utility |
|---|---|
| `font-[100]` | `font-thin` |
| `font-[200]` | `font-extralight` |
| `font-[300]` | `font-light` |
| `font-[400]` | `font-normal` |
| `font-[500]` | `font-medium` |
| `font-[600]` | `font-semibold` |
| `font-[700]` | `font-bold` |
| `font-[800]` | `font-extrabold` |
| `font-[900]` | `font-black` |

**Spacing / sizing (`w-[*]`, `h-[*]`, `p-[*]`, `m-[*]`, `gap-[*]`, etc.)**

> ⚠️ **v4 is formula-based, not a fixed lookup table.** Since Tailwind v4, the spacing scale is generated as `calc(var(--spacing) * N)` (default `--spacing: 0.25rem` = 4px) for **any** integer or half-integer `N` — not just a curated set of "common" numbers. This is a change from v3, which only shipped a fixed set of named steps (`0, 0.5, 1, 1.5, 2, 2.5, 3, 3.5, 4, 5, 6, 7, 8, 9, 10, 11, 12, 14, 16, 20, 24, 28, 32, 36, 40, 44, 48, 52, 56, 60, 64, 72, 80, 96`) with real gaps between them (e.g. no `28.5`, nothing between `12` and `14`).
>
> **The rule that actually matters:** in a v4 project, any arbitrary px value that is a clean multiple of 4 (or 2, for the `.5` steps) has a matching numeric utility, *even if that exact number isn't in the table below*. Compute it — don't just look it up:
>
> ```
> N = px / 4        (e.g. 104px / 4 = 26 → w-26)
> if N is a whole number or ends in .5 → utility exists, convert
> otherwise → keep as arbitrary (e.g. 15px, 45px)
> ```
>
> In a v3 project, only check against the fixed step list above — v3 does **not** support arbitrary `N` the way v4 does. If Dimension 2 detected v3, cross-reference against the named steps only.
>
> The table below lists common reference points, not the exhaustive set of valid values.

| px value | Tailwind scale | rem |
|---|---|---|
| 4px | `1` | 0.25rem |
| 8px | `2` | 0.5rem |
| 12px | `3` | 0.75rem |
| 16px | `4` | 1rem |
| 20px | `5` | 1.25rem |
| 24px | `6` | 1.5rem |
| 32px | `8` | 2rem |
| 40px | `10` | 2.5rem |
| 48px | `12` | 3rem |
| 64px | `16` | 4rem |
| 80px | `20` | 5rem |
| 96px | `24` | 6rem |
| 128px | `32` | 8rem |
| 160px | `40` | 10rem |
| 192px | `48` | 12rem |
| 224px | `56` | 14rem |
| 256px | `64` | 16rem |
| 288px | `72` | 18rem |
| 320px | `80` | 20rem |
| 384px | `96` | 24rem |
