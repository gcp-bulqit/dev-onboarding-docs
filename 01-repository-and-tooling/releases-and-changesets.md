# Releases and Changesets

## Paths

| Item | Location |
|------|----------|
| CLI | Root `package.json` — `@changesets/cli`, scripts `changeset`, `version-packages`, `release` |

## What the scripts achieve

- **`pnpm changeset`** — Interactive CLI to add a **changeset file** describing semver bumps and changelog notes per affected package.
- **`pnpm version-packages`** — Consumes changesets, bumps `package.json` versions, dedupes changelog entries.
- **`pnpm release`** — Typically `turbo run build` then `changeset publish` (exact publish targets depend on org setup).

**Synopsis:** Changesets give a **reviewable, auditable** path from merged PRs to versioned npm (or private registry) releases without manual version editing.

Release policy (who publishes, which branch) is a team process—align with maintainers before running publish commands.
