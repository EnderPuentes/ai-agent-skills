# Check: `inline-jsx-constants`

## What to flag

- A variable holding **JSX** (`const Foo = <div>...</div>`) that is **only** used once—or used as a stand-in for a tiny fragment that could live **inline** in the return tree.

Preferred: inline the JSX in `return (` or extract a **real** `function Subname()` subcomponent when reuse or clarity warrants (per project UI rules: JSX subcomponents use `function`, not `const` arrows—see repo `ui-component-spec`).

## Severity / confidence

- **medium** for needless indirection that hurts scanability.
- **high** if the pattern hides conditional logic or hooks incorrectly (rare—raise only with evidence).

## Reference anti-pattern

Holding a block of JSX in a `const` then placing it under the return—integrate into the tree or promote to a proper `function` component.

## False positives

- JSX assigned for **arrays** (`items.map`-style data) or **legitimate** reuse in multiple branches.
