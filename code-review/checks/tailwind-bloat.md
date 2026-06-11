# Check: `tailwind-bloat`

## What to flag

- `className` strings with **many** utilities (rough guide: **≥ ~8** distinct tokens) especially when wrapping **shadcn/system** components that already take `className` and merge variants.
- Repeated **responsive stacks** (`sm: md: lg:`) on leaf nodes where a parent already defines layout/spacing.
- **Duplication** of the same utilities on siblings that could share a wrapper (only when obvious in the diff).

## Severity / confidence

- **high**: Conflicting or redundant utilities that suggest misunderstanding of the child API (e.g. full padding/typography on a `Button` that already sets them via variants)—only when the diff makes it obvious.
- **medium**: Long `className` on simple wrappers; likely trimmable after checking the design system.
- **low**: Slightly verbose but readable; optional cleanup.

**Confidence**: lower when the imported component’s defaults are **not** in the diff—say “verify against component defaults.”

## False positives

- One-off complex layouts that truly need many utilities.
- `cva()` variant strings (review variant design, not raw count).
