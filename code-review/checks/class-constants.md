# Check: `class-constants`

## What to flag

- `const` / `let` whose value is primarily a **Tailwind class string** (or array/object of class strings) used to avoid repeating markup—**especially** when the constant is only referenced once or twice.

Project expectation: **repeat the class string** at the call site when duplication is minor; extract a **named component** or **CVA-based** variant when abstraction is warranted—not a bare string constant.

## Severity / confidence

- **medium** for new `const styles = "..."` or `const X_CLASSES = "..."` in TSX.
- **low** if the constant is shared across **many** sites in the same file and a component split is a larger refactor—suggest follow-up.

## False positives

- `cva(...)` definitions (those are intentional variant systems).
- Non-CSS string constants (URLs, keys, copy).
