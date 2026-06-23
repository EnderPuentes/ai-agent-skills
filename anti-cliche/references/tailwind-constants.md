# Tailwind & Style Constants Clichés

## 1. The Flywheel Constants Cliché

**The Cliché**: Creating named constants for individual Tailwind strings under the guise of maintainability. The constants don't add meaning — they just rename strings that already describe themselves.

```ts
// ❌ constants/styles.ts
export const CARD_BASE = 'rounded-lg border bg-white shadow-sm'
export const CARD_HOVER = 'hover:shadow-md hover:border-gray-300'
export const CARD_PADDING = 'p-6'
export const CARD_HEADER = 'flex items-center justify-between mb-4'
export const BUTTON_PRIMARY = 'bg-blue-600 text-white px-4 py-2 rounded-md'
export const BUTTON_DISABLED = 'opacity-50 cursor-not-allowed'

// Usage
<div className={cn(CARD_BASE, CARD_HOVER, CARD_PADDING)}>
```

**Why it's wrong**: `CARD_BASE` tells you nothing `'rounded-lg border bg-white shadow-sm'` doesn't already say. You've added a layer of indirection with zero semantic gain. Now you have to jump to the constants file to understand what the component looks like.

**The Fix**: Write Tailwind inline. The class string is self-documenting.

```ts
// ✅
<div className="rounded-lg border bg-white shadow-sm hover:shadow-md p-6">
```

---

## 2. The `cn()` Overuse Cliché

**The Cliché**: Using `clsx` / `cn` to concatenate two completely static strings that never vary.

```ts
// ❌
className={cn(WRAPPER_BASE, WRAPPER_PADDING)}

// ❌ Even without constants:
className={cn('flex flex-col', 'gap-4 p-6')}
```

**The Fix**: `cn()` / `clsx()` is for **conditional** class logic. Static classes go inline.

```ts
// ✅ Use cn() only when there's actual conditional logic
className={cn('flex flex-col gap-4 p-6', isActive && 'ring-2 ring-blue-500')}

// ✅ Fully static? Just write it
className="flex flex-col gap-4 p-6"
```

---

## 3. The `STYLES` Object Cliché

**The Cliché**: Grouping constants into a namespace object to feel "organized."

```ts
// ❌
const STYLES = {
  wrapper: 'flex flex-col gap-4',
  header: 'text-lg font-semibold text-gray-900',
  body: 'text-sm text-gray-600',
  footer: 'flex justify-end gap-2 mt-4',
}
```

**Why it's wrong**: You've created a parallel shadow-stylesheet. Now styles live in two places: the component and the object. Tailwind IntelliSense and refactoring tools can't reliably follow references through object keys.

**The Fix**: Keep styles in the JSX. Use `cva` (class-variance-authority) *only* when you have genuine variants with multiple states.

```ts
// ✅ Legitimate use of cva — real variants, real states
const button = cva('rounded-md font-medium transition-colors', {
  variants: {
    intent: {
      primary: 'bg-blue-600 text-white hover:bg-blue-700',
      ghost: 'bg-transparent text-gray-700 hover:bg-gray-100',
    },
    size: {
      sm: 'px-3 py-1.5 text-sm',
      md: 'px-4 py-2 text-base',
    },
  },
})
```

---

## 4. Token Constants That Duplicate Tailwind's Design System

**The Cliché**: Re-defining Tailwind's spacing/color scale as custom constants.

```ts
// ❌
export const SPACING_4 = 'p-4'
export const SPACING_6 = 'p-6'
export const TEXT_MUTED = 'text-gray-500'
export const TEXT_DANGER = 'text-red-600'
```

**The Fix**: Tailwind already is your design token system. Use `tailwind.config.ts` to extend the theme if you need custom tokens. Don't duplicate it with JS constants.

```ts
// ✅ tailwind.config.ts
theme: {
  extend: {
    colors: {
      brand: { DEFAULT: '#2563eb', dark: '#1d4ed8' }
    }
  }
}
// Then just use: className="text-brand"
```

---

## 5. The Premature Abstraction into `ui/` Primitives

**The Cliché**: Before having real reuse evidence, creating `ui/Typography.tsx`, `ui/Stack.tsx`, `ui/Grid.tsx` that are just HTML elements with a few fixed Tailwind classes.

```tsx
// ❌
const Stack = ({ children, gap = 4 }) => (
  <div className={`flex flex-col gap-${gap}`}>{children}</div>
)
// Used twice, both times with gap=4
```

**The Fix**: Write the JSX directly. Extract into a component only when the same pattern appears 3+ times *and* includes logic or meaningful variance beyond a single class.
