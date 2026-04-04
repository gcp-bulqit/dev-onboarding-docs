# Quality and testing

## Paths

| Concern | Path / command |
|---------|----------------|
| Shared ESLint | `packages/eslint-config/` (`exports`: `./node`, `./next`) |
| Root lint | `pnpm lint` → Turbo runs each package's `lint` script |
| Format | Root `package.json` `format` → Prettier over `**/*.{ts,tsx,md}` |
| Type check | `pnpm check-types` → `tsc` across packages |
| Web unit/integration | `apps/web/package.json` — `test`, `test:unit`, `test:integration` (Vitest) |
| Test setup | `apps/web/src/test-setup.ts` |
| Vitest unit config | `apps/web/vitest.config.mjs` |
| Vitest integration config | `apps/web/vitest.integration.config.mjs` |

## What each layer achieves

- **ESLint (`@bulqit/eslint-config`)** — Enforces React/Next/TypeScript/import-sort rules consistently. The shared config loads `eslint-plugin-react-compiler`, but the web app disables it in `apps/web/eslint.config.mjs` (`'react-compiler/react-compiler': 'off'`).
- **`pnpm lint` / `turbo run lint`** — Runs lint in parallel only where needed based on Turbo's dependency graph and cache.
- **`pnpm check-types`** — Runs `tsc` across packages that define `check-types`; catches cross-package type breakage.
- **Vitest (`apps/web`)** — `test:unit` exercises pure components and utils; `test:integration` hits DB/API (see **`AGENTS.md`**: integration tests **truncate** tables — use only a disposable dev DB). 429 test files on staging.

## TypeScript strictness

`apps/web/tsconfig.json` sets `"strict": false` with `"strictNullChecks": true`. Other strict-mode flags (`noImplicitAny`, `strictFunctionTypes`) are not enabled.

## Coverage

Vitest coverage uses the `v8` provider with `text`, `json`, and `html` reporters. No coverage thresholds are configured in either `vitest.config.mjs` or `vitest.integration.config.mjs`.

## Test setup

`apps/web/src/test-setup.ts` configures `@testing-library/react` with `testIdAttribute: 'datatest-id'` (non-standard; the conventional attribute is `data-testid`).

## E2E

`e2e-cypress/` contains 5 spec files:

| File | Scope |
|------|-------|
| `memberCheckout.cy.ts` | Member checkout flow |
| `memberDashboard.cy.ts` | Member dashboard |
| `memberOnboarding.cy.ts` | Member onboarding |
| `vendor.cy.ts` | Vendor sign-up (currently skipped pending AddressAutocompleteField deploy) |
| `vendorDashboard.cy.ts` | Vendor dashboard |

## Skipped tests

3 unit tests are currently skipped with `it.skip`:

| File | Test |
|------|------|
| `completeAddOnJobAction.test.ts:187` | should throw error when user is not signed in |
| `job-card.test.tsx:113` | should render completed date |
| `job-card.test.tsx:330` | should handle "Not scheduled" as completedDate |
