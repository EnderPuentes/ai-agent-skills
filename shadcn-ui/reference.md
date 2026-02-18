# shadcn/ui — Reference & Official Documentation

This file complements the shadcn-ui skill with official documentation links and extra context for indexing.

## Official documentation (indexable)

- **Introduction**: https://ui.shadcn.com/docs  
- **Installation**: https://ui.shadcn.com/docs/installation  
- **Create new project**: https://ui.shadcn.com/create  
- **Components (overview)**: https://ui.shadcn.com/docs/components  
- **Directory (community components)**: https://ui.shadcn.com/docs/directory  
- **Tailwind v4**: https://ui.shadcn.com/docs/tailwind-v4  
- **Dark mode**: https://ui.shadcn.com/docs/dark-mode  
- **Theming**: See “Theming” in main docs and `components.json` `tailwind.baseColor` / CSS variables.

## Framework-specific installation

- **Next.js**: Use `npx shadcn@latest create` or `init`; select Next.js. App Router is supported; set `rsc: true` in `components.json` when using Server Components.
- **Vite**: https://ui.shadcn.com/docs/installation/vite — Tailwind via `@tailwindcss/vite`, path alias `@/*`, then `shadcn init`.
- **Manual**: https://ui.shadcn.com/docs/installation/manual — Add Tailwind, path aliases, `cn` helper, and create `components.json` by hand if needed.

## Tailwind v4 + shadcn

- We use **Tailwind v4 only**; no `tailwind.config.js`.
- In `components.json`, leave `tailwind.config` as `""`.
- Global styles: one CSS file with `@import "tailwindcss"` and, if needed, `@theme { ... }` for design tokens.
- shadcn theming is done with **CSS variables** in that same global CSS file (e.g. `--background`, `--foreground`, `--primary`). These work with Tailwind v4’s `@theme` if you map or override tokens there.

## Component list (CLI names for `shadcn add`)

Use these with: `pnpm dlx shadcn@latest add <name>`.

**Layout & structure**: accordion, aspect-ratio, card, collapsible, resizable, separator, scroll-area  

**Forms & input**: button, checkbox, form, input, input-otp, label, radio-group, select, slider, switch, textarea  

**Navigation & menus**: breadcrumb, command, context-menu, dropdown-menu, menubar, navigation-menu, pagination, tabs  

**Overlays & feedback**: alert, alert-dialog, dialog, drawer, hover-card, popover, sheet, tooltip  

**Data & state**: avatar, badge, calendar, carousel, chart, progress, skeleton, sonner, table, toast (deprecated; use sonner)  

**Toggle**: toggle, toggle-group  

**Dependencies**: Adding some components (e.g. calendar, carousel, chart, form) will install peer deps (react-day-picker, embla, recharts, react-hook-form, zod, etc.). Check CLI output.

## components.json schema

- **Schema URL**: https://ui.shadcn.com/schema.json  
- Key fields: `style`, `rsc`, `tailwind.config`, `tailwind.css`, `tailwind.baseColor`, `tailwind.cssVariables`, `aliases.components`, `aliases.utils`, `aliases.ui`, `aliases.lib`, `aliases.hooks`, `iconLibrary`.

## Primitives (Radix UI)

shadcn/ui components are built on **Radix UI** primitives. For low-level behavior or accessibility details, see:

- **Radix UI**: https://www.radix-ui.com/primitives

## Icons

- **Lucide** (default in many examples): https://lucide.dev  
- Configure in `components.json` with `iconLibrary` (e.g. `lucide`).

## Related skills

- **tailwind-css**: Tailwind v4 setup, `@theme`, `@import "tailwindcss"`, Next.js PostCSS, no config file. Use when configuring Tailwind or theming.
