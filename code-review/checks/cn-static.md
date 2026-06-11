# Check: `cn-static`

## What to flag

- `cn(\`...\`)` or `cn("...")` where **all** arguments are **static** string literals with **no** variables, **no** conditional expressions, and **no** `className` prop merge.

Preferred: put the string directly on `className="..."` (or template literal if needed for readability only when it includes expressions).

## Keep `cn()` when

- Merging **user/forwarded** `className` with defaults.
- **Conditional** classes: `cn(base, isActive && "active", className)`.
- **Dynamic** segments from props/state.

## Severity / confidence

- **low**–**medium** by default (ergonomics and consistency).
- **high** only if static `cn` is used pervasively in new code in the same file (noise / convention break).

## False positives

- `cn()` wrapping **only** to satisfy a typing quirk—rare; mention **low** confidence if unsure.
