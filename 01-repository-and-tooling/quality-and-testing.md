# Quality and testing

## Paths

| Concern | Path / command |
|---------|----------------|
| Shared ESLint | `packages/eslint-config/` (`exports`: `./node`, `./next`) |
| Root lint | `pnpm lint` → Turbo runs each package’s `lint` script |
| Format | Root `package.json` `format` → Prettier over `*.{ts,tsx,md}` |
| Web unit/integration | `apps/web/package.json` — `test`, `test:unit`, `test:integration` (Vitest) |

## What each layer achieves

- **ESLint (`@bulqit/eslint-config`)** — Enforces React/Next/TypeScript/import-sort rules consistently; `eslint-plugin-react-compiler` and Next plugin catch framework-specific mistakes.
- **`pnpm lint` / `turbo run lint`** — Runs lint in parallel only where needed based on Turbo’s dependency graph and cache.
- **`pnpm check-types`** — Runs `tsc` across packages that define `check-types`; catches cross-package type breakage.
- **Vitest (`apps/web`)** — `test:unit` exercises pure components and utils; `test:integration` hits DB/API (see **`AGENTS.md`**: integration tests **truncate** tables—use only a disposable dev DB).

## E2E

Cypress is referenced in root `README.md` via `CYPRESS_BASE_URL`; the `e2e-cypress` workspace may hold suites (see `pnpm-workspace.yaml`).
