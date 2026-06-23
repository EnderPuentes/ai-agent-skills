# AI Agent Skills

Repository containing reusable skills for AI agents, following the [skills.sh](https://skills.sh) specification.

## Available Skills

### Tooling

Git workflow, commits, and code review skills.

#### anti-cliche

Enforce strict anti-cliché rules across prose, code, UI/UX, and Next.js/Tailwind/React architecture (2026 Standards).

**Location**: `anti-cliche/`

**Use when**: You want to avoid tropes, banned words, overengineering, UI clichés, excessive abstraction, prop-drilling, legacy React 18 patterns, or generic AI-generated code tells.

**Features**:

- 2026 React/Next.js Architecture defaults (React Compiler, use() hook, URL State)
- Tailwind v4 best practices
- AI Code Tells prevention (obvious comments, passive-aggressive error handling)
- UI/UX Post-AI Boom checks
- Banned writing tropes and phrases

#### conventional-commits

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
- **reference.md**: Official spec, commit types, SemVer correlation, tooling — indexable

**Docs**: [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)

#### code-review

Audit the git diff against `develop` for UI/React review themes—Tailwind bloat, `cn()` misuse, class constants, inline JSX constants, SRP boundaries, legacy-in-system imports—grading severity and confidence. Text-only audit; no auto-fixes.

**Location**: `code-review/`

**Use when**: Reviewing a feature branch vs `develop`, running a structured UI/React code review audit, or when the user asks for a develop-based code review without applying fixes.

**Features**:

- Modular checks in `checks/` (tailwind-bloat, cn-static, class-constants, inline-jsx-constants, single-responsibility, legacy-in-system)
- Severity and confidence grading per finding
- Mandatory output shape (summary, findings, clean pass, follow-ups)

**Author**: Bruno Balderrama ([bmbalderrabano@gmail.com](mailto:bmbalderrabano@gmail.com)) (MIT, included with permission)

#### split-commit

Analyze local git changes and return a copy-paste PowerShell script to split uncommitted work into multiple logical commits. The agent does not commit for the user.

**Location**: `split-commit/`

**Use when**: Splitting uncommitted changes into multiple commits, planning commit batches, or when the user wants a ready-to-run PowerShell sequence for staged commits.

**Features**:

- Read-only git inspection; no `git add` / `git commit` / `git push` by the agent
- Commit message rules aligned with `.githooks/commit-msg` (branch-aware)
- Copy-paste PowerShell output with per-file `git add` and conventional commit messages

**Author**: Bruno Balderrama ([bmbalderrabano@gmail.com](mailto:bmbalderrabano@gmail.com)) (MIT, included with permission)

### Productivity

Planning, design stress-testing, and decision-making workflows.

#### clinical-tone

Enforce a strictly objective, clinical, and non-condescending communication style.

**Location**: `clinical-tone/`

**Use when**: You want the AI to communicate like a senior engineer without sycophancy, apologies, over-enthusiasm, or didactic lectures.

**Features**:

- Strips away AI filler ("Great idea!", "I apologize", "As an AI")
- Focuses on fact-based critique and direct answers
- Prevents "sandwich feedback" (praise wrapping criticism)

#### grill-me

Interview the user relentlessly about a plan or design until reaching shared understanding. Walks each branch of the design tree one question at a time, with a recommended answer per question.

**Location**: `grill-me/`

**Use when**: Stress-testing a plan, getting grilled on a design, refining architecture before implementation, or when the user mentions "grill me".

**Features**:

- One question at a time; resolve decision dependencies sequentially
- Recommended answer included with each question
- Explores the codebase when a question can be answered there instead of asking the user

**Source**: [mattpocock/skills](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) (MIT)

### Styling

CSS frameworks and component libraries.

#### tailwind-css

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

#### shadcn-ui

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

### UI & animation

Component development and motion.

#### gsap

Install and use GSAP (GreenSock Animation Platform) for high-performance JavaScript animations. Covers core utilities, React integration, and popular plugins.

**Location**: `gsap/`

**Use when**: Creating complex animations, scroll-based effects, UI transitions, SVG animations, or when the user mentions GSAP, GreenSock, or ScrollTrigger.

**Features**:

- Core utilities: Tween, Timeline, Eases
- React integration with `@gsap/react` and `useGSAP`
- Popular plugins: ScrollTrigger, Flip, Observer, etc.
- Best practices for performance and FOUC prevention
- **reference.md**: Official GSAP docs, core API, plugins, React integration — indexable

**Docs**: [GSAP](https://gsap.com/docs/v3/)

#### storybook

Install and use Storybook for developing UI components in isolation. Colocated stories in `system/` directory, CSF format, args, decorators, play functions, and testing.

**Location**: `storybook/`

**Use when**: Building component libraries, design systems, documenting UI components, testing components in isolation, or when the user mentions Storybook, stories, component development, or UI workshop.

**Features**:

- Installation for Next.js, Vite, React, Vue, Angular, Svelte
- **system/** structure: components in folders with colocated `*.stories.tsx`
- Component Story Format (CSF 3): meta, stories, args
- Decorators, parameters, play functions for interaction tests
- Vitest addon, accessibility, visual testing
- **reference.md**: Official Storybook docs, frameworks, addons, testing — indexable

**Docs**: [Storybook](https://storybook.js.org/docs)

### Language & validation

TypeScript, schemas, and API documentation.

#### typescript

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

#### zod

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

#### jsdoc

Write and maintain API documentation using JSDoc comments colocated with JavaScript/TypeScript code. Covers comment rules, core tags, and CLI generation.

**Location**: `jsdoc/`

**Use when**: Documenting functions/classes/modules, standardizing `@param` and `@returns`, generating API docs, or when the user mentions JSDoc tags like `@todo`.

**Features**:

- Comment block rules (`/** ... */`) and placement guidelines
- Core tags (`@param`, `@returns`, `@constructor`, `@todo`)
- Common examples for functions and constructor-style docs
- CLI docs generation flow (`jsdoc file.js`)
- **reference.md**: Official JSDoc docs links and command-line/config references — indexable

**Docs**: [JSDoc](https://jsdoc.app/)

### Performance

Web performance auditing and optimization.

#### pagespeed-insights

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
- **reference.md**: Official PageSpeed, Lighthouse, Core Web Vitals, optimization guides — indexable

**Docs**: [PageSpeed Insights](https://developers.google.com/speed/docs/insights/v5/about?hl=es-419)

### SEO & AEO

Discoverability for search engines and AI assistants. Three format-specific skills — install the one that matches the task.

#### sitemap

Create and maintain XML sitemaps per sitemaps.org for search engine crawl discovery.

**Location**: `sitemap/`

**Use when**: Adding `sitemap.xml`, sitemap index, Next.js `sitemap.ts`, `lastmod`, `changefreq`, robots.txt `Sitemap:` directive, or when the user mentions sitemap or URL discovery.

**Features**:

- XML protocol 0.9 (`urlset`, `sitemapindex`, tags, limits)
- Entity escaping, placement rules, FAQ highlights
- Next.js `MetadataRoute.Sitemap` pattern
- vs JSON-LD and llms.txt
- **reference.md**: sitemaps.org protocol, FAQ, XSD, Google — indexable

**Docs**: [sitemaps.org](https://www.sitemaps.org/protocol.html)

#### json-ld

Add and validate JSON-LD structured data with schema.org for SEO rich results. Based on W3C JSON-LD 1.1 and Google Search Central.

**Location**: `json-ld/`

**Use when**: Structured data, schema markup, `application/ld+json`, Organization, WebSite, Article, Product, FAQ, BreadcrumbList, JSON-LD, schema.org, or rich snippets.

**Features**:

- Core JSON-LD syntax, Next.js embedding, common schema.org types
- Validation workflow (Playground, Rich Results Test)
- Google SEO guidelines and common mistakes
- **reference.md**: W3C, schema.org, Google Search Central — indexable

**Docs**: [JSON-LD 1.1](https://www.w3.org/TR/json-ld11/) · [schema.org](https://schema.org/)

#### llms-txt

Create `llms.txt` per llmstxt.org for AI assistants and AEO/GEO.

**Location**: `llms-txt/`

**Use when**: Adding `llms.txt`, `llms-full.txt`, AI-readable site maps, agent context, AEO, GEO, or when the user mentions llms.txt or llmstxt.

**Features**:

- Official spec format (H1, blockquote, file lists, `## Optional`)
- Static and dynamic Next.js delivery
- `.md` mirrors, llms-ctx; vs sitemap and JSON-LD
- **reference.md**: llmstxt.org, tools, examples — indexable

**Docs**: [llmstxt.org](https://llmstxt.org/)

## Structure

Each skill follows the standard skills.sh structure:

```
skill-name/
├── SKILL.md          # Required: metadata + instructions
├── reference.md      # Optional: official docs, deep reference (tailwind-css, shadcn-ui)
└── LICENSE.md        # Optional
```

Skills live at the repository root (no nested group folders) so `npx skills add --skill <name>` works out of the box.

## Installation

### Tooling

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill anti-cliche
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill conventional-commits
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill code-review
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill split-commit
```

### Productivity

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill clinical-tone
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill grill-me
```

### Styling

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill tailwind-css
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill shadcn-ui
```

### UI & animation

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill gsap
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill storybook
```

### Language & validation

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill typescript
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill zod
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill jsdoc
```

### Performance

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill pagespeed-insights
```

### SEO & AEO

```bash
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill sitemap
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill json-ld
npx skills add https://github.com/EnderPuentes/ai-agent-skills --skill llms-txt
```

For more information about using skills in your agent, follow the [skills.sh integration guide](https://skills.sh).

## Contributing

When adding new skills, ensure they follow:

- Skills.sh specification
- Conventional Commits for commit messages
- Clear, concise documentation
- Practical examples
