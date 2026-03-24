# Git hooks (Lefthook)

## Paths

| Item | Path |
|------|------|
| Install | Root `package.json` `prepare` → `lefthook install` (skipped if `SKIP_PREPARE=true`) |
| Config | `lefthook.yml` |

## What runs (from `lefthook.yml`)

| Hook | Commands | Purpose |
|------|----------|---------|
| **pre-commit** | `pnpm turbo lint --continue` on `*.{js,ts,jsx,tsx,mjs,cjs}` | Catches lint errors before commit; `stage_fixed: true` re-stages ESLint fixes when applicable. |
| **pre-commit** | `pnpm turbo check-types --continue` on `*.{ts,tsx}` (with larger Node heap) | Typecheck changed TS before commit. |
| **post-merge** | `pnpm install` | Refreshes deps after pulling merges. |
| **post-merge** | `pnpm build:packages` | Rebuilds shared packages so dependents compile after lockfile or package changes. |

**Synopsis:** Hooks keep **`lint` + `check-types`** green locally so CI failures are less frequent; post-merge steps reduce “works on my machine” after pulls.
