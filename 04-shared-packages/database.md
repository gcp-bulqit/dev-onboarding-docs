# `packages/database`

## Paths

| Item | Path |
|------|------|
| Package | `packages/database/` |
| Schemas | `packages/database/src/schemas/` |
| Migrations | `packages/database/migrations/` |

## Exports (synopsis)

`package.json` **exports** include:

- **`./orm`** — Drizzle ORM entry.
- **`./schemas`** — Table definitions and relations consumed across apps.
- **`./instances/node`**, **`./instances/serverless`**, **`./instances/http`**, **`./instances/integrations`** — Preconfigured DB clients for Node workers, Vercel serverless, HTTP drivers, integration tests.

## Behavior synopsis

Central **Postgres schema** and **migration history** for the whole product. Workers call **`getInstance({ connectionString })`**; Next uses serverless-friendly pools where appropriate. All domain tables (members, vendors, jobs, notifications queue, messaging, etc.) live here—**single source of truth** for Drizzle types.

Follow **`AGENTS.md`**: migrations via **`pnpm drizzle generate`** only.
