# Tailwind CSS v4 — Reference & Official Documentation

This file complements the tailwind-css skill with theme details and links to the official docs for indexing and deeper reference.

## Official documentation (indexable)

- **Installation**
  - Next.js: https://tailwindcss.com/docs/installation/framework-guides/nextjs
  - Vite: https://tailwindcss.com/docs/installation/framework-guides/vite
  - Tailwind CLI: https://tailwindcss.com/docs/installation/tailwind-cli
- **Core concepts**
  - Theme variables: https://tailwindcss.com/docs/theme
  - Adding custom styles: https://tailwindcss.com/docs/adding-custom-styles
  - Styling with utility classes: https://tailwindcss.com/docs/styling-with-utility-classes
- **Compatibility**: https://tailwindcss.com/docs/compatibility
- **Editor setup** (e.g. Cursor/VS Code): https://tailwindcss.com/docs/editor-setup

## Theme: advanced options

### Overriding an entire namespace

Set the namespace to `initial` and then define only the variables you want:

```css
@theme {
  --color-*: initial;
  --color-white: #fff;
  --color-primary: #3f3cbb;
  --color-midnight: #121063;
}
```

### Using a fully custom theme

Disable default theme and use only your tokens:

```css
@theme {
  --*: initial;
  --spacing: 4px;
  --font-body: Inter, sans-serif;
  --color-lagoon: oklch(0.72 0.11 221.19);
  --color-coral: oklch(0.74 0.17 40.24);
}
```

### Generating all CSS variables (static)

Use `@theme static` so all theme variables are always emitted:

```css
@theme static {
  --color-primary: var(--color-red-500);
  --color-secondary: var(--color-blue-500);
}
```

### Sharing theme across projects

Put shared tokens in a separate CSS file and import it:

```css
/* packages/brand/theme.css */
@theme {
  --*: initial;
  --font-body: Inter, sans-serif;
  --color-primary: oklch(0.62 0.21 260);
}

/* app/globals.css */
@import "tailwindcss";
@import "../brand/theme.css";
```

## Using theme variables elsewhere

- **In custom CSS**: `var(--color-gray-700)`, `var(--text-base)`, `var(--radius-xl)`.
- **In arbitrary values**: `rounded-[calc(var(--radius-xl)-1px)]`.
- **In JavaScript**: `getComputedStyle(document.documentElement).getPropertyValue("--shadow-xl")`.

## Default theme (summary)

Tailwind’s default theme provides:

- **Fonts**: `--font-sans`, `--font-serif`, `--font-mono`
- **Colors**: Full palettes for red, orange, amber, yellow, lime, green, emerald, teal, cyan, sky, blue, indigo, violet, purple, fuchsia, pink, rose, slate, gray, zinc, neutral, stone, plus `--color-black`, `--color-white`
- **Breakpoints**: `--breakpoint-sm` (40rem), `--breakpoint-md` (48rem), `--breakpoint-lg` (64rem), `--breakpoint-xl` (80rem), `--breakpoint-2xl` (96rem)
- **Spacing**: `--spacing` (0.25rem) and spacing scale
- **Typography**: `--text-*`, `--font-weight-*`, `--tracking-*`, `--leading-*`
- **Radius**: `--radius-xs` through `--radius-4xl`
- **Shadows**: `--shadow-*`, `--inset-shadow-*`, `--drop-shadow-*`
- **Easing**: `--ease-in`, `--ease-out`, `--ease-in-out`
- **Animations**: `--animate-spin`, `--animate-ping`, `--animate-pulse`, `--animate-bounce` (with keyframes)

Full default theme: see `node_modules/tailwindcss/theme.css` or the [theme variables docs](https://tailwindcss.com/docs/theme).

## Functional utilities (@utility with --value())

For utilities that take a value, use `@utility` with `--value()`:

- Theme: `--value(--tab-size-*)`
- Bare type: `--value(integer)`, `--value(percentage)`
- Literal: `--value("inherit", "initial", "unset")`
- Arbitrary: `--value([integer])`, `--value([length])`

Example:

```css
@theme {
  --tab-size-github: 8;
}
@utility tab-* {
  tab-size: --value(--tab-size-*, integer, [integer]);
}
```

## No config file (reminder)

Tailwind v4 does **not** use:

- `tailwind.config.js`
- `tailwind.config.ts`
- `tailwind.config.mjs`
- `content: []`
- `theme: { extend: { ... } }`
- `plugins: []` in a JS config

Configuration is done only in CSS via `@import "tailwindcss"`, `@theme`, `@layer`, `@utility`, and `@custom-variant`, plus the build plugin (`@tailwindcss/vite` or `@tailwindcss/postcss`).
