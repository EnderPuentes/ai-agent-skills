---
name: jsdoc
description: "Write and maintain JSDoc comments for JavaScript and TypeScript APIs: functions, classes, modules, parameters, returns, examples, and TODO annotations. Use when documenting code, generating API docs, setting JSDoc conventions, or when the user mentions JSDoc tags like @param, @returns, or @todo."
---

# JSDoc

## Overview

JSDoc is a documentation generator for JavaScript. You write documentation comments directly above code, and JSDoc parses those comments to generate API documentation.

**Core principle**: Document the public API clearly and keep comments colocated with code so docs stay accurate as implementation changes.

---

## Comment rules

- JSDoc blocks must start with `/**` (exactly two stars after the slash).
- Blocks that start with `/*`, `/***`, or more stars are ignored by the JSDoc parser.
- Place the comment immediately before the symbol being documented (function, class, method, module, etc.).
- Prefer documenting public and reusable APIs first.

---

## Quick start

```js
/**
 * Adds two numbers.
 * @param {number} a - First number.
 * @param {number} b - Second number.
 * @returns {number} Sum of the two numbers.
 */
function add(a, b) {
  return a + b;
}
```

---

## Core tags

| Tag | Use |
|-----|-----|
| `@param {Type} name - description` | Document function parameters |
| `@returns {Type} description` | Document return value |
| `@constructor` | Mark a function as a constructor |
| `@todo text` | Track pending tasks in docs |

Use additional tags as needed for your project (`@example`, `@throws`, `@deprecated`, `@see`, etc.).

---

## Common documentation patterns

### Functions

```js
/**
 * Formats a user's full name.
 * @param {string} firstName - User first name.
 * @param {string} lastName - User last name.
 * @returns {string} The formatted full name.
 */
function formatName(firstName, lastName) {
  return `${firstName} ${lastName}`;
}
```

### Constructor-style functions

```js
/**
 * Represents a book.
 * @constructor
 * @param {string} title - Book title.
 * @param {string} author - Book author.
 */
function Book(title, author) {
  this.title = title;
  this.author = author;
}
```

### TODO annotations

```js
/**
 * @todo Implement pagination support.
 * @todo Add validation for empty input.
 */
function listItems() {}
```

---

## Generate documentation

Basic CLI usage:

```bash
jsdoc book.js
```

This generates HTML docs in an `out/` directory by default.

You can also pass multiple source files:

```bash
/path/to/jsdoc src/a.js src/b.js
```

You may provide a config file for CLI options and template customization.

---

## Best practices

- Keep descriptions short, specific, and focused on behavior.
- Document parameter and return types consistently.
- Include `@todo` for known gaps that must be tracked.
- Update comments whenever API signatures change.
- Document exported/public modules first, then internal helpers as needed.
- Keep examples executable and realistic.

---

## Common mistakes

- Using `/* ... */` instead of `/** ... */` for JSDoc blocks.
- Writing comments far from the symbol they describe.
- Missing `@param` entries for function inputs.
- Describing implementation details instead of API behavior.
- Letting comments drift from current code after refactors.

---

## Additional resources

- [reference.md](reference.md) — Official JSDoc docs links for quick lookup.
- **Official docs**: https://jsdoc.app
