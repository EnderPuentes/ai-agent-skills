---
name: code-review
description: Audits the git diff against develop (not main) for UI/React review themes—Tailwind bloat, cn() misuse, class constants, inline JSX constants, SRP boundaries, legacy-in-system imports—grading severity and confidence. Use when the user asks for a develop-based code review audit without auto-fixes.
---

# Code review

**Author**: Bruno Balderrama ([bmbalderrabano@gmail.com](mailto:bmbalderrabano@gmail.com)). Included in this collection with the author's permission.

These modular checks are also wired into the **Standards** axis of the [`review`](../.agents/skills/review/SKILL.md) skill (`/fly-review`) when the diff touches UI/React files. Use this skill standalone for a develop-based UI audit; use `review` for the full two-axis Standards + Spec pass.

## Goal

Produce a **text-only audit** of changes versus **`develop`** (never `main` as the comparison base). The user decides what to fix; **do not** apply edits unless they explicitly ask.

## Modular checks

Apply **every** checklist file below against the diff. When adding a new theme, add `checks/<id>.md` and a bullet here.

| ID                      | Topic                                                              |
| ----------------------- | ------------------------------------------------------------------ |
| `tailwind-bloat`        | [checks/tailwind-bloat.md](checks/tailwind-bloat.md)               |
| `cn-static`             | [checks/cn-static.md](checks/cn-static.md)                         |
| `class-constants`       | [checks/class-constants.md](checks/class-constants.md)             |
| `inline-jsx-constants`  | [checks/inline-jsx-constants.md](checks/inline-jsx-constants.md)   |
| `single-responsibility` | [checks/single-responsibility.md](checks/single-responsibility.md) |
| `legacy-in-system`      | [checks/legacy-in-system.md](checks/legacy-in-system.md)           |

Read each linked file before grading that category.

## Step 1 — Scope the diff (repo root)

**Base branch:** `develop` (prefer `origin/develop` if local `develop` is missing or stale).

Suggested commands (read-only):

```powershell
git fetch origin develop 2>$null
git branch --show-current
git log -1 --oneline origin/develop 2>$null; if (-not $?) { git log -1 --oneline develop }
```

**Primary range (commits on this branch vs develop):**

```powershell
git diff origin/develop...HEAD
```

If `origin/develop` is unavailable, use `develop...HEAD`.

**Include uncommitted changes** in the audit when relevant:

```powershell
git diff origin/develop
```

(or `git diff develop`)

Use the combined picture: **symmetric range** for “what the branch adds,” **two-arg diff** when local working tree matters. If both are empty, say so and stop.

Large changesets: use `git diff --stat` / path-scoped `git diff origin/develop...HEAD -- path` to stay thorough.

## Step 2 — Grade each finding

For **each** issue, output:

- **File** (and line if visible in diff hunk)
- **Check** (ID from the table)
- **Severity**: `low` | `medium` | `high` (user impact / maintenance cost if ignored)
- **Confidence**: `low` | `medium` | `high` (how sure you are this is a real problem, not a false positive)
- **Note**: one or two sentences; optional **suggestion** (still no edits)

**Severity guide**

- **high**: Violates an explicit project rule, likely bugs, or entrenches architecture debt (e.g. legacy under `system/`).
- **medium**: Noise, inconsistency, or extra maintenance; should usually fix before merge.
- **low**: Style preference or uncertain redundancy; mention briefly.

**Confidence guide**

- **high**: Clear pattern in the hunk (e.g. `cn("a b c")` with no variables).
- **medium**: Likely issue; parent component not in diff.
- **low**: Needs design context; flag as “verify manually.”

## Step 3 — Output shape (mandatory)

Plain text (or markdown) in this order:

1. **Summary**: 2–4 bullets (branch, base, scope used, rough counts).
2. **Findings**: grouped by **severity** (high → medium → low) or by **file**—pick one and stay consistent.
3. **Clean pass**: list check IDs with no issues as “OK” or “not applicable.”
4. **Optional follow-ups**: items that need runtime/design verification.

No auto-fixes, no commits, no PR creation.

## Constraints

- **Do not** use `main` as the diff base unless the user explicitly overrides.
- **Do not** treat this audit as blocking CI; it complements human review.
- Prefer **evidence from the diff**; when inferring from incomplete context, lower **confidence**.
- Be **extremely succinct** and prioritize low token consumption.
