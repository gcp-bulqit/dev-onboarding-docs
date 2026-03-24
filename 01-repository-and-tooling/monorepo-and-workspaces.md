# Monorepo and workspaces

## Paths

| Item | Path |
|------|------|
| Workspace globs | `pnpm-workspace.yaml` |
| Turbo pipeline (tasks, caching, task graph) | `turbo.json` |
| Root orchestration scripts | `package.json` |
| Node version pin | `.nvmrc` |

## What the configuration achieves

- **`pnpm-workspace.yaml`** — Declares that every package under `apps/*` and `packages/*` (and `e2e-cypress` if present) is part of one workspace so dependencies can use `workspace:*` and installs are hoisted/deduped efficiently.
- **`turbo.json`** — Defines named tasks (`build`, `dev`, `lint`, `check-types`, `test`, etc.), their **dependencies** between packages, inputs, and outputs so **Turbo** can parallelize work and cache artifacts; changing a file only rebuilds downstream dependents.
- **Root `package.json` scripts** — `pnpm build` runs `turbo run build`; `pnpm dev` runs `turbo run dev`; `pnpm lint`, `pnpm check-types`, `pnpm test` fan out the same way. Worker deploy scripts (`deploy:worker-*`) prune the monorepo and invoke `flyctl` for specific apps.

## Commands (synopsis)

| Command | Effect |
|---------|--------|
| `pnpm install` | Resolves the full workspace lockfile and links local packages. |
| `pnpm dev` | Starts every app’s `dev` script that Turbo knows about (often the web app + any watch processes). |
| `pnpm dev --filter=bulqit-website` | Limits dev to the Next app package name. |
| `pnpm build` | Cached, dependency-ordered production builds. |
| `pnpm check-types` | TypeScript across packages that expose a `check-types` task. |
| `pnpm typegen` | Runs Sanity type generation via `@bulqit/sanity-cms`. |

See also root `README.md` and `AGENTS.md`.
