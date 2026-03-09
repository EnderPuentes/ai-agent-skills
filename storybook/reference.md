# Storybook — Reference & Official Documentation

This file complements the storybook skill with official documentation links and sections for indexing.

## Official documentation (indexable)

- **Docs home**: https://storybook.js.org/docs
- **Get started**: https://storybook.js.org/docs/get-started
- **Installation**: https://storybook.js.org/docs/get-started/install
- **Writing stories**: https://storybook.js.org/docs/writing-stories
- **Writing docs**: https://storybook.js.org/docs/writing-docs
- **Writing tests**: https://storybook.js.org/docs/writing-tests
- **Configure**: https://storybook.js.org/docs/configure
- **Sharing**: https://storybook.js.org/docs/sharing
- **Essential features**: https://storybook.js.org/docs/essentials
- **Addons**: https://storybook.js/addons
- **FAQ**: https://storybook.js.org/docs/faq

## Frameworks

- **Next.js (Vite)**: https://storybook.js.org/docs/get-started/frameworks/nextjs-vite — Recommended for Next.js
- **Next.js (Webpack)**: https://storybook.js.org/docs/get-started/frameworks/nextjs
- **React + Vite**: https://storybook.js.org/docs/get-started/frameworks/react-vite
- **Vue 3 + Vite**: https://storybook.js.org/docs/get-started/frameworks/vue3-vite
- **Angular**: https://storybook.js.org/docs/get-started/frameworks/angular
- **SvelteKit**: https://storybook.js.org/docs/get-started/frameworks/sveltekit
- **Svelte + Vite**: https://storybook.js.org/docs/get-started/frameworks/svelte-vite
- **Web Components**: https://storybook.js.org/docs/get-started/frameworks/web-components-vite

## Core concepts

- **Stories**: Rendered component states; one story = one configuration
- **CSF (Component Story Format)**: ES module standard; default export = meta, named exports = stories
- **Args**: Props/inputs; editable via Controls panel
- **Decorators**: Wrappers for stories (layout, providers, padding)
- **Parameters**: Static metadata for addons (backgrounds, layout, etc.)
- **Play function**: Async code that runs after story render; used for interaction tests

## Configuration

- **main.ts**: `stories`, `addons`, `framework`, `staticDirs`, `viteFinal`, `webpackFinal`
- **preview.ts**: `decorators`, `parameters`, `globalTypes`
- **Story loading**: Glob patterns, directory config, `titlePrefix`, custom `files`

## Testing

- **Vitest addon**: https://storybook.js.org/docs/writing-tests/integrations/vitest-addon
- **Test runner**: https://storybook.js.org/docs/writing-tests/integrations/test-runner
- **Interaction testing**: https://storybook.js.org/docs/writing-tests/interaction-testing
- **Accessibility**: https://storybook.js.org/docs/writing-tests/accessibility-testing
- **Visual testing**: https://storybook.js.org/docs/writing-tests/visual-testing
- **CI**: https://storybook.js.org/docs/writing-tests/in-ci

## Addons (common)

- **@storybook/addon-essentials**: Controls, Actions, Viewport, Backgrounds, Toolbars, Measure, Outline
- **@storybook/addon-a11y**: Accessibility checks
- **@storybook/addon-vitest**: Run stories as Vitest tests
- **@storybook/addon-docs**: Autodocs, MDX

## Project requirements (from install guide)

- Node.js 20+
- npm 10+ / pnpm 9+ / Yarn 4+
- Vite 5+ or Webpack 5+
- TypeScript 4.9+ (optional)
- Next.js 14+, Vue 3+, React 18+, etc. (framework-specific)
