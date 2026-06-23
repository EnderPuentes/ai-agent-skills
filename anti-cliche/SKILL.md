---
name: anti-cliche-agent
description: >
  Enforce strict anti-cliché rules across prose, code, UI/UX, and Next.js/Tailwind/React architecture (2026 Standards).
  Use this skill whenever the user wants to avoid tropes, banned words, overengineering, UI clichés,
  excessive abstraction, prop-drilling, legacy React 18 patterns, or generic AI-generated code tells.
  Also triggers on: 'anti-cliche', 'banned words', 'avoid tropes', 'no clichés', 'clean architecture',
  'no overengineering', 'avoid AI writing', 'Next.js best practices', 'React 19'.
  When in doubt, use this skill — it applies to nearly all code review, UI critique, and writing tasks.
globs:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
  - "**/*.css"
  - "**/*.md"
  - "**/*.mdx"
  - "app/**/*"
  - "components/**/*"
  - "lib/**/*"
  - "actions/**/*"
  - "hooks/**/*"
  - "styles/**/*"
  - "constants/**/*"
  - "types/**/*"
---

# Anti-Cliché Agent — Next.js / Tailwind / React Edition (2026 Standards)

**Core Principle**: Every banned pattern substitutes form for content, or complexity for value.
The fix is always: be specific, be cohesive, show consequence, respect what already exists, and embrace modern platform defaults.

For detailed reference on each domain, load the relevant file from `references/`:
- Architecture & overengineering → `references/architecture.md`
- Tailwind & style constants → `references/tailwind-constants.md`
- React components & composition → `references/components.md`
- Server Components & Server Actions → `references/server.md`
- UI/UX patterns → `references/ui-ux.md`
- Prose & writing → `references/prose.md`

---

## Quick-Reference Banned Patterns

### React 19+ & Architecture
- **Legacy React Clichés**: Do not use `useMemo`, `useCallback`, or `React.memo` (rely on the React Compiler). Do not use `forwardRef` (use the `ref` prop). Do not use `useContext` (favor the `use()` hook).
- **Types & Naming**: Do not use `any` (use `unknown` with narrowing). Follow standard ecosystem naming conventions (e.g., PascalCase for React components) without forcing arbitrary naming structures unless defined by the repository.
- **AI Code Tells**: Do not write obvious inline comments (e.g., `// Check if user exists`).
- **Passive-Aggressive Errors**: Massive `try/catch` blocks that just do `console.error` and return `null`. Instead, use explicit error returns (like the Result pattern: `{ success: true, data }` or `{ success: false, error }`) and let unexpected system errors bubble up to Error Boundaries.
- Barrel files (`index.ts`) that re-export everything from a folder just to look organized.
- `utils/helpers.ts` god files — name things by what they do, not that they "help".

### Tailwind v4 & Constants
- **The `@apply` Directive**: Never use `@apply`. Use utility classes directly in the markup or compose them via CVA for variants.
- **Over-Configured Variants**: Use CVA (Class Variance Authority) for legitimate component variants, but do not create massive JS config files for single-use elements. 
- **Flywheel Constants**: `const STYLES = { wrapper: 'flex flex-col gap-4' }` — do not split one inline class string into named constants to feel "organized". Use the `cn()` utility ONLY for actual conditional classes.

### Components & Composition
- `<Box>`, `<Flex>`, `<Wrapper>`, `<Container>` that only pass props through without adding logic.
- Components extracted for "reuse" that are used exactly once.
- **Prop Drilling & Overuse of Props**: Do not pass props down multiple levels (prop-drilling). Avoid bloated components that accept dozens of props. Use React composition (`children`) or colocate state instead.
- **JSX Variables (JSX en Constantes)**: Do not assign JSX elements to regular constants or variables (e.g., `const icon = <div>Icon</div>`). If a chunk of JSX needs to be extracted, it MUST become a proper functional component using the `function` keyword (e.g., `function Icon() { return <div>Icon</div>; }`). Do NOT use arrow functions (`const Component = () => {}`) for component declarations.
- `interface Props extends React.HTMLAttributes<HTMLDivElement>` on every component by default.

### Server Components & Actions
- **State instead of URL**: Using `useState` for search queries, tabs, or pagination. This is a Single Page App cliché. Use URL `searchParams`.
- `"use client"` on every file by default without a real interactivity reason.
- Fetching data in a client component when a Server Component would be simpler.
- Server Actions that do nothing but call a function that could be inline.

### UI/UX (Post-AI Boom)
- **"Slap an AI chat on it"**: Do not put a generic floating chatbox or a "✨ Generate" button on every screen. Prioritize predictive, invisible, and contextual AI integrations.
- **Generic Startup UI**: Hero sections with gradient text + floating 3D blobs + meaningless glassmorphism.
- Low-contrast decorative text (gray-400 on white) for "clean" aesthetics.
- Every interactive element animating on scroll with `framer-motion` by default.

### Prose & Writing
- **Banned words**: tapestry, landscape, delve, foster, underscore, showcase, pivotal, crucial, profound, palpable, raw, visceral, primal, bone-deep.
- **Banned tags**: softly, quietly, carefully, flatly, evenly.
- **Banned filler**: "for a long moment", "time stretched", "light spills".
- **Banned physical tells**: jaw tightens, throat bobs, breath catches, eyes darken.

---

## Decision Protocol

Before writing any code, component, constant, or prose block:

1. **Does this already exist in the codebase?** Use it. Don't add a parallel pattern.
2. **Is this used more than once?** If not, don't extract it.
3. **Does this constant add semantic meaning, or just name a string?** If the latter, inline it.
4. **Is `"use client"` here because of actual state/event, or just habit?** Remove if habit.
5. **Is this component a wrapper with no logic?** Delete it, compose inline.
6. **Does this UI element communicate information or just decorate?** Remove decoration.
7. **Could a Server Component fetch this data instead?** Yes → use Server Component.
8. **Are you assigning JSX to a variable?** Turn it into a functional component instead.
9. **Are you passing a prop more than 2 levels deep?** Rethink and use React composition.
10. **Are you using `useState` for something that should be a URL parameter?** Move it to the URL.
11. **Are you writing `useCallback` / `useMemo`?** Delete them and let the React Compiler handle it.

---

## The Corrective Framework

| Cliché | Ask Instead |
|--------|-------------|
| Abstract factory for one use | What's the simplest function that does this? |
| Flywheel style constant | Does naming this string add meaning not in the class itself? |
| Prop-drilling wrapper | Can I colocate this, or use React composition? |
| `"use client"` by default | What specific browser API or event triggers this? |
| Generic hero UI | What specific user problem does this layout solve? |
| Floating AI "✨" button | How can I integrate this AI feature directly into the user's natural workflow? |
| `useState` for tabs/filters | Can I read this from `searchParams`? |
| `try/catch` that swallows errors | Should this component fail fast and trigger an Error Boundary? |
| Banned prose word | What is the concrete, observable thing I'm trying to say? |
