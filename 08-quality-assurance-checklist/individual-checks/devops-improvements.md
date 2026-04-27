## DevOps & CI-CD Improvements

> Last updated: <!-- TODO: update date when editing -->

This document collects actionable suggestions to improve the reliability and speed of the monorepo’s Continuous Integration and broader DevOps posture.

---

### 1. GitHub Actions (`.github/workflows/ci.yml`)

1. **Cache dependencies & browsers**
   - Enable **pnpm** cache via `actions/setup-node` (`cache: 'pnpm'`) or `pnpm/action-setup@v2` with `run_install`.
   - Cache Playwright browsers:
     ```yaml
     - uses: actions/cache@v4
       with:
         path: ~/.cache/ms-playwright
         key: ${{ runner.os }}-playwright-${{ hashFiles('**/pnpm-lock.yaml') }}
     ```
2. **Fail-fast installs**
   - Use `pnpm install --frozen-lockfile`.
3. **Job concurrency**
   ```yaml
   concurrency:
     group: ${{ github.workflow }}-${{ github.ref }}
     cancel-in-progress: true
   ```
4. **Split jobs** (`lint → type-check → unit-test → build → e2e`) using `needs:`.
5. **Turborepo remote caching**
   - Uncomment `TURBO_TOKEN` / `TURBO_TEAM` to drastically cut build times.
6. **Reuse build output**: upload `.next` as an artifact; download in the e2e job.
7. **Health-check endpoint**: expose `/healthz` and wait on it instead of raw port.
8. **Node version matrix**: `20`, `22`.
9. **Neon parent branch**: derive from `github.sha`/`ref_name` instead of hard-coded `main`.

---

### 2. Monorepo Hygiene

- Add **Dependabot** or **Renovate** for automated dependency & action updates.
- Ensure each workspace package defines standard `build`, `lint`, `test` scripts for consistent `turbo` tasks.
- Add `.npmrc` with `shared-workspace-lockfile=true` for a single root lock-file.
- Centralise environment variables in a root `.env.example` & map GitHub secrets to environments (`ci`, `prod`, …).
- Introduce pre-commit hooks (`husky`, `lefthook`) to run `lint`/`format` locally.

---

### 3. Observability & Quality Gates

- Upload coverage to Codecov/Coveralls and enforce a minimum threshold.
- Run **Lighthouse CI** during build or on Preview deployments.
- Upload **Sentry source-maps** using `@sentry/cli` for accurate stack traces.
- Add vulnerability scanning (Trivy / Snyk) step.

---

### 4. Containerisation & Deployment

- Create a reusable composite action to **build, scan and push Docker images** for apps like `apps/temporal-workers`.
- Generate and attach **SBOMs** (`cosign sbom`, `cyclonedx`) to releases.

---

### 5. Documentation & DX

- Document local test flow in `docs/ci.md` for new contributors (`act`, `turbo run …`).
- Use **Actions Summary** to publish Playwright reports & coverage HTML.

---

#### Quick-win trio (often 50-70 % faster pipelines)

1. pnpm & Playwright caching
2. Turborepo remote cache (`TURBO_TOKEN`)
3. Split early-fail jobs (lint, types)
