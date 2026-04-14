---
name: storybook
description: Install and use Storybook for developing UI components in isolation. Covers CSF (Component Story Format), colocated stories in system/ directory, installation for Next.js/Vite/React, writing stories with args, decorators, play functions, and testing. Use when building component libraries, design systems, documenting UI components, testing components in isolation, or when the user mentions Storybook, stories, component development, or UI workshop.
---

# Storybook

## Overview

Storybook is a **frontend workshop** for building UI components and pages in isolation. It helps develop and share hard-to-reach states and edge cases without running the whole app. Use it for UI development, testing, and documentation.

**Core concepts**: Stories (rendered component states), CSF (Component Story Format), args (props/inputs), decorators (wrappers), play functions (interaction tests).

**Requirements**: Node.js 20+, npm 10+ or pnpm 9+ or Yarn 4+, Vite 5+ or Webpack 5+, TypeScript 4.9+ (optional).

---

## Installation

Reference: [Official install guide](https://storybook.js.org/docs/get-started/install)

| Action | CLI command |
|--------|--------------|
| New or existing project | `create storybook@latest` |
| Specific framework | `create storybook@latest --type <framework>` |
| Dev server | `run storybook` |
| Build | `run build-storybook` |

Invoke via your package manager (see [docs](https://storybook.js.org/docs/get-started/install)). The CLI detects the framework (Next.js, Vite, React, Vue, Angular, Svelte, etc.). For manual selection: `--type nextjs`, `react`, `vue3`, `angular`, `svelte`, `sveltekit`, etc.

**Recommended**: Use `@storybook/nextjs-vite` for Next.js projects (faster than Webpack). Use `@storybook/react-vite` for Vite + React.

Dev server: typically port 6006.

---

## Project structure: system directory

Place components in a **`system/`** directory, with each component in its own folder. Colocate the story file alongside the component — this keeps documentation and tests next to the source and follows Storybook best practices.

```
system/
├── Button/
│   ├── Button.tsx
│   └── Button.stories.tsx
├── Card/
│   ├── Card.tsx
│   └── Card.stories.tsx
├── Dialog/
│   ├── Dialog.tsx
│   └── Dialog.stories.tsx
```

Configure `.storybook/main.ts` to load stories from `system/`:

```ts
import type { StorybookConfig } from '@storybook/your-framework';

const config: StorybookConfig = {
  framework: '@storybook/your-framework',
  stories: ['../system/**/*.stories.@(js|jsx|mjs|ts|tsx)'],
};

export default config;
```

---

## Component Story Format (CSF)

Stories use **CSF 3**: a default export (meta) and named exports (stories).

### Meta (default export)

```tsx
import type { Meta } from '@storybook/your-framework';
import { Button } from './Button';

const meta = {
  component: Button,
  title: 'System/Button',  // optional; auto-derived from path if omitted
} satisfies Meta<typeof Button>;

export default meta;
```

### Stories (named exports)

```tsx
import type { StoryObj } from '@storybook/your-framework';

type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: {
    label: 'Button',
    primary: true,
  },
};

export const Secondary: Story = {
  args: {
    ...Primary.args,
    primary: false,
    label: 'Secondary',
  },
};
```

**Args** map to component props. They appear in the Controls panel for live editing.

---

## Writing stories

### Basic story with args

```tsx
export const Default: Story = {
  args: {
    label: 'Click me',
    disabled: false,
  },
};
```

### Custom render

Use `render` when the default (component + args) is not enough:

```tsx
export const InAlert: Story = {
  args: { label: 'Confirm' },
  render: (args) => (
    <Alert>
      <Button {...args} />
    </Alert>
  ),
};
```

### Play function (interaction tests)

Simulate user interactions and assert outcomes:

```tsx
import { expect } from 'storybook/test';

export const OpensDialog: Story = {
  play: async ({ canvas, userEvent }) => {
    await userEvent.click(canvas.getByRole('button', { name: 'Open' }));
    await expect(canvas.getByRole('dialog')).toBeInTheDocument();
  },
};
```

### Decorators

Wrap stories (e.g. padding, theme, providers):

```tsx
const meta = {
  component: Button,
  decorators: [
    (Story) => (
      <div style={{ padding: '3em' }}>
        <Story />
      </div>
    ),
  ],
} satisfies Meta<typeof Button>;
```

### Parameters

Static metadata for addons (backgrounds, layout, etc.):

```tsx
const meta = {
  component: Button,
  parameters: {
    backgrounds: { values: [{ name: 'dark', value: '#333' }] },
  },
} satisfies Meta<typeof Button>;
```

---

## Reusing stories across components

Import story args from child components to compose parent stories:

```tsx
import * as ButtonStories from '../Button/Button.stories';

export const WithButtons: Story = {
  args: {
    buttons: [
      { ...ButtonStories.Primary.args },
      { ...ButtonStories.Secondary.args },
    ],
  },
};
```

---

## Configuration

### main.ts

- **stories**: Glob patterns for story files.
- **addons**: e.g. `@storybook/addon-essentials`, `@storybook/addon-a11y`.
- **framework**: e.g. `@storybook/nextjs-vite`, `@storybook/react-vite`.
- **staticDirs**: Static assets (e.g. `['../public']`).

### preview.ts

Global decorators, parameters, and CSS:

```ts
import type { Preview } from '@storybook/react';
import '../app/globals.css';

const preview: Preview = {
  decorators: [
    (Story) => (
      <div style={{ fontFamily: 'system-ui' }}>
        <Story />
      </div>
    ),
  ],
  parameters: {
    layout: 'centered',
  },
};

export default preview;
```

---

## Testing

- **Render tests**: Every story is a smoke test — it passes if it renders without error.
- **Interaction tests**: Use `play` with `expect` from `storybook/test`.
- **Vitest addon**: `storybook add @storybook/addon-vitest` — runs stories as Vitest tests (invoke via package manager; see [docs](https://storybook.js.org/docs/writing-tests/integrations/vitest-addon)).
- **Accessibility**: Enable `@storybook/addon-a11y` for automated a11y checks.
- **Visual testing**: Use Chromatic or snapshot tests for visual regression.

---

## Best practices

- **Colocate stories** with components in `system/<ComponentName>/`.
- **Use args** instead of hardcoding props; enables Controls and reuse.
- **One story per meaningful state** (default, loading, error, empty, etc.).
- **Reuse child story args** when composing parent stories.
- **Use `play`** for interaction tests instead of manual E2E when possible.
- **Keep stories focused** — one component per story file; use `render` for composition.

---

## Common mistakes

- **Stories in separate folder**: Prefer colocation (`Component.stories.tsx` next to `Component.tsx`).
- **Missing `component` in meta**: Required for autodocs and Controls; use `satisfies Meta<typeof Component>`.
- **Hardcoded props**: Use `args` so Controls work and stories are reusable.
- **Wrong framework import**: Use `@storybook/nextjs-vite`, `@storybook/react-vite`, etc., not generic `@storybook/react`.
- **Glob not matching system/**: Ensure `stories` in `main.ts` includes `../system/**/*.stories.*`.

---

## Additional resources

- [reference.md](reference.md) — Official Storybook docs links, frameworks, addons, testing.
- **Official**: https://storybook.js.org/docs
- **Writing stories**: https://storybook.js.org/docs/writing-stories
- **Testing**: https://storybook.js.org/docs/writing-tests
