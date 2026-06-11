---
name: split-commit
description: Analyze local git changes and return a copy-paste split-commit PowerShell command sequence. Use when the user wants to split uncommitted work into multiple logical commits without the agent committing for them.
disable-model-invocation: true
---

# Git split commits

Collaboration with **Bruno Balderrama** ([bmbalderrabano@gmail.com](mailto:bmbalderrabano@gmail.com)).

Goal: split **current local changes** into multiple logical commits by producing a **single PowerShell script** the user runs themselves.

## Constraints (mandatory)

- **Do not** run `git add`, `git commit`, or `git push` for the user. Only read-only inspection commands below (and any other non-mutating git queries you need).
- **Do not** stage, commit, or push on the user’s behalf.
- Return **only one** final block: a **copy-paste-ready PowerShell** sequence (comments + commands).

## Step 1 — Inspect (repository root)

From the repository root, run and use the output for grouping and message style:

```powershell
git branch --show-current
git status --short
git log -12 --oneline
git diff
```

If the change set is large, you may also use `git diff --stat` or path-limited `git diff -- path` to reason about batches—still **no** `git add` / `git commit` / `git push`.

## Commit messages — `.githooks/commit-msg` (mandatory)

**Source of truth:** `.githooks/commit-msg`. Every proposed `git commit -m` **must** pass that hook for the **current branch** (from `git branch --show-current`).

| Branch | Allowed first line (non-empty, trimmed) |
|--------|----------------------------------------|
| `main` | Merge-style line (`Merge branch…`, `Merge remote-tracking branch…`, `Merge commit…`, `Merge pull request…`) **or** `hotfix(scope):` subject |
| `develop` | **Only** merge-style lines (same patterns as above). No `feat:` / `fix:` / etc. on `develop` |
| **other** | Conventional Commits: `type(scope): subject` or `type: subject` — space after `:`; subject non-empty; types: `feat`, `fix`, `hotfix`, `chore`, `test`, `docs`, `style`, `ci`; **≤ 120 characters** on the first line |

If the user is on `develop` and needs multiple commits, say clearly they must use a **feature branch** (or accept `--no-verify`, not recommended). Do not emit conventional `git commit -m` lines for `develop` unless the message is merge-shaped.

Imperative mood and consistency with `git log -12 --oneline` still apply where conventional commits are used.

## Step 2 — Plan batches

1. Group changed paths into **logical** commits (feature vs fix vs docs vs tests vs chore, or by subsystem).
2. Draft **commit messages** that satisfy **`.githooks/commit-msg`** for the current branch (see table above).

## Step 3 — Emit the PowerShell sequence

**Hook compliance (agent-side only):** Read `.githooks/commit-msg` and the **Commit messages** table above. Every `git commit -m "…"` line you output must satisfy that hook for the current branch (allowed types on feature branches are only: `feat`, `fix`, `hotfix`, `chore`, `test`, `docs`, `style`, `ci` — e.g. **`refactor` is rejected**). Do **not** emit helpers, temp files, or `Assert-*`; the user validates by running commits normally.

**Output shape (always use this layout):**

- Assume the user is **already at the repo root**.
- First lines: start-clean comment + `git reset HEAD` (omit `git reset HEAD` if the repo has no commits yet; say so in the comment).
- For **each** batch, in order:
  - One comment: `# <n>) <short title>` (increment `n`).
  - **`git add "path"`** for **each** file in that batch (quote paths; one `git add` per line).
  - **`git commit -m "exact first line"`** — literal double-quoted string; imperative mood; `type(scope): subject` or `type: subject`; ≤ 120 characters.
- After all batches, **always** end with:

```powershell
git status --short
git log -12 --oneline
```

**Example structure** (shape must match; use **`feat`/`fix`/`chore`/…** per the hook — not `refactor`):

```powershell
# Start clean (omit if repo has no commits yet)
git reset HEAD
# 1) Shared route URL helper
git add "src/lib/navigation/route-params.ts"
git commit -m "chore(lib): add replaceRouteSearchParams for list route queries"
# 2) System components: single pattern for updating search params
git add "src/components/system/search/index.tsx"
git add "src/components/system/search/ui.tsx"
git add "src/components/system/filters/dropdown/index.tsx"
git add "src/components/system/filters/search-button/index.tsx"
git add "src/components/system/sorts/dropdown/index.tsx"
git commit -m "chore(system): use shared route param updates in search and filters"
# 3) Header + client list-filter hooks (param naming)
git add "src/components/layouts/header/index.tsx"
git add "src/components/layouts/header/hooks/use-list-filters.tsx"
git add "src/components/layouts/header/hooks/use-assistants-filters.tsx"
git commit -m "chore(layouts): align list header and filter hooks with route param names"
# 4) Spaces list: URL-driven filters, parse-url, tests
git add "src/lib/spaces/parse-url.ts"
git add "src/app/(authenticated)/spaces/hooks/filters.tsx"
git add "src/app/(authenticated)/spaces/partials/list.tsx"
git add "__tests__/spaces-list-filters.test.tsx"
git commit -m "chore(spaces): route-based list filters and simplify URL parse"
git status --short
git log -12 --oneline
```

The assistant’s final reply should contain **nothing else** except that one PowerShell block (optional one-line intro if required; prefer **only** the block).

## Edge cases

- **No changes**: emit a minimal script that only runs `git status --short` and `git log -12 --oneline`, with a comment that there is nothing to split.
- **Untracked files**: include them in an appropriate batch with `git add "path"` in the **user-run** script only (you still do not run `git add` yourself).
- **Single logical commit**: one batch is fine; still follow the same script structure.
