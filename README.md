# AI Agent Skills

Repository containing reusable skills for AI agents, following the [skills.sh](https://skills.sh) specification.

## Available Skills

### conventional-commits

Generate and validate commit messages following the Conventional Commits specification.

**Location**: `conventional-commits/`

**Use when**: Creating git commits, writing commit messages, reviewing commit history, generating changelogs, or when working with semantic versioning.

**Features**:

- Complete Conventional Commits specification reference
- Quick reference tables for commit types and structure
- Common mistakes and best practices
- Examples for all commit types
- Breaking change handling
- Semantic versioning correlation

**Note**: This skill is built from the official documentation at https://www.conventionalcommits.org/en/v1.0.0/

### pagespeed-insights

Audit web pages for performance optimization following PageSpeed Insights guidelines. Acts as a performance auditor that identifies bad practices and guides developers to achieve excellent PageSpeed scores.

**Location**: `pagespeed-insights/`

**Use when**: Analyzing page performance, optimizing web applications, reviewing performance metrics, implementing Core Web Vitals improvements, or when the user mentions page speed, performance optimization, Lighthouse scores, or Core Web Vitals.

**Features**:

- Complete PageSpeed Insights guidelines reference
- Core Web Vitals thresholds and best practices
- Common performance issues and solutions
- Lab and field data interpretation
- Performance optimization checklist
- Accessibility and SEO best practices
- Code examples for common optimizations

**Note**: This skill is built from the official documentation at https://developers.google.com/speed/docs/insights/v5/about?hl=es-419

### tailwind-css

Set up and use **Tailwind CSS v4 only** with Next.js (App Router), Vite, or CLI. No `tailwind.config.js`—theme and config via `@theme` in CSS. Best practices from official docs.

**Location**: `tailwind-css/`

**Use when**: Styling with Tailwind, configuring Tailwind, installing Tailwind, utility classes, `@theme`, `@layer`, `@utility`, or when the user mentions Tailwind CSS or Tailwind v4.

**Features**:

- Installation: Next.js (App Router, 15+), Vite, Tailwind CLI
- Theme: `@theme` only (extend, override, namespaces, keyframes, inline)
- Custom styles: `@layer base` / `@layer components`, `@utility`, `@custom-variant`
- Arbitrary values and properties
- Quick reference and common mistakes
- **reference.md**: Official doc links, advanced theme options, default theme summary (indexable)

**Docs**: [Tailwind CSS v4](https://tailwindcss.com/docs)

### shadcn-ui

Install and use shadcn/ui with **Tailwind v4** and Next.js (App Router), Vite, or manual setup. Configure `components.json`, add components via CLI, theming with CSS only (no Tailwind config file).

**Location**: `shadcn-ui/`

**Use when**: Adding shadcn/ui, installing shadcn, using Button, Card, Dialog, Form, or any shadcn component, or when the user mentions shadcn, shadcn/ui, or Radix UI with Tailwind.

**Features**:

- Quick start (`shadcn create`) and existing-project setup (Next.js, Vite)
- `components.json` and path aliases
- **Full list of official components** with CLI names and short descriptions (accordion, alert, button, card, dialog, form, sonner, table, etc.)
- Theming with CSS variables and `@theme` (Tailwind v4)
- Common patterns and mistakes
- **reference.md**: Official doc links, Tailwind v4 + shadcn, component list, schema (indexable)

**Docs**: [shadcn/ui](https://ui.shadcn.com/docs)

### typescript

Use TypeScript for type-safe JavaScript: types, interfaces, generics, narrowing, tsconfig, modules, and strict mode.

**Location**: `typescript/`

**Use when**: Writing or reviewing TypeScript, configuring tsconfig.json, defining types or interfaces, generics, type inference, or when the user mentions TypeScript, TS, or type checking.

**Features**:

- Everyday types, narrowing, functions, object types
- Type manipulation: generics, keyof, typeof, mapped types, conditional types, utility types
- Classes, modules
- tsconfig.json and strict mode
- **reference.md**: Official doc links (Handbook, Reference, TSConfig, Cheat Sheets) — indexable

**Docs**: [TypeScript](https://www.typescriptlang.org/docs)

### zod

Define and use Zod schemas for TypeScript-first validation with static type inference (z.infer). Primitives, objects, arrays, unions, refinements, transform, parse, safeParse.

**Location**: `zod/`

**Use when**: Validating input, parsing API data, form validation with react-hook-form, or when the user mentions Zod, schema validation, or z.infer.

**Features**:

- Primitives, strings (email, url, uuid, datetime), numbers, objects, arrays, tuples, unions, discriminated unions
- Refinements, transform, pipe
- z.infer, z.input, z.output; parse / safeParse, ZodError
- React Hook Form integration (zodResolver)
- **reference.md**: Official Zod docs and API sections (zod.dev, API, llms.txt), Zod Mini, ecosystem — indexable

**Docs**: [Zod](https://zod.dev)

## Structure

Each skill follows the standard skills.sh structure:

```
skill-name/
├── SKILL.md          # Required: metadata + instructions
├── reference.md      # Optional: official docs, deep reference (tailwind-css, shadcn-ui)
└── LICENSE.md        # Optional
```

## Installation

### Install conventional-commits skill

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill conventional-commits
```

### Install pagespeed-insights skill

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill pagespeed-insights
```

### Install tailwind-css skill

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill tailwind-css
```

### Install shadcn-ui skill

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill shadcn-ui
```

### Install typescript skill

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill typescript
```

### Install zod skill

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill zod
```

For more information about using skills in your agent, follow the [skills.sh integration guide](https://skills.sh).

## Contributing

When adding new skills, ensure they follow:

- Skills.sh specification
- Conventional Commits for commit messages
- Clear, concise documentation
- Practical examples
