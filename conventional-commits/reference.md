# Conventional Commits — Reference & Official Documentation

This file complements the conventional-commits skill with official documentation links for indexing.

## Official documentation (indexable)

- **Specification v1.0.0**: https://www.conventionalcommits.org/en/v1.0.0/
- **FAQ**: https://www.conventionalcommits.org/en/v1.0.0/#faq
- **Examples**: https://www.conventionalcommits.org/en/v1.0.0/#examples

## Specification summary

- **Format**: `<type>[optional scope]: <description>`
- **Footer**: `BREAKING CHANGE:` or `Token: value`
- **Breaking change indicator**: `!` after type/scope

## Commit types (standard)

- **feat**: New feature (SemVer MINOR)
- **fix**: Bug fix (SemVer PATCH)

## Commit types (common extensions)

- **build**, **chore**, **ci**, **docs**, **hotfix**, **perf**, **refactor**, **style**, **test**
- **revert**: Revert a previous commit

## Semantic Versioning correlation

- **fix** / **hotfix** → PATCH (1.0.0 → 1.0.1)
- **feat** → MINOR (1.0.0 → 1.1.0)
- **BREAKING CHANGE** / **!** → MAJOR (1.0.0 → 2.0.0)

## Tooling (ecosystem)

- **standard-version**: https://github.com/conventional-changelog/standard-version
- **semantic-release**: https://github.com/semantic-release/semantic-release
- **commitlint**: https://commitlint.js.org/
- **conventional-changelog**: https://github.com/conventional-changelog/conventional-changelog
- **husky + commitlint**: Pre-commit validation

## Footer tokens

- **BREAKING CHANGE**: Indicates breaking change (MAJOR bump)
- **Refs**: Reference to commits, issues, PRs
- **Co-authored-by**: https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/creating-a-commit-with-multiple-authors

## Best practices (reminder)

- Imperative mood: "add" not "added" or "adds"
- Description ≤ 72 characters (or 120 per team)
- Scope in parentheses: `feat(auth):`
- Colon and space after type/scope: `feat: `
