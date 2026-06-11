# Check: `legacy-in-system`

## What to flag

Any import in **`src/components/system/**`** (or paths aliased as `@/components/system/...`) that resolves under **`components/legacy/`** or matches `@/components/legacy`.

Rationale: **legacy** is slated for removal; **system** is the supported design-system surface. New coupling blocks deprecation.

## Severity / confidence

- **high** for new or expanded legacy imports in `system/` files.
- **medium** if the diff only touches an existing line—still call out **debt**.

## Evidence

Quote the import line and both path segments (`system` + `legacy`).

## False positives

- Imports from **`system`** into **legacy** (discouraged but opposite direction—optional note, lower severity).
- Non-component paths; adjust if the repo adds exceptions (document in this file when they exist).
