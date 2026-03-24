# Zoho sync worker

## Paths

| Item | Path |
|------|------|
| Entry | `apps/worker-zoho-sync/src/index.ts` |
| Sync cycles | `apps/worker-zoho-sync/src/cycles` (or `./cycles` import) |
| Environment | `apps/worker-zoho-sync/src/environment.ts` |
| Client | `packages/zoho-client/` — `ZohoClient` |

## Behavior synopsis — `index.ts`

1. **`Sentry.init`** if configured.
2. **`getInstance`** — DB connection string from env.
3. **`new ZohoClient({ clientId, refreshToken, clientSecret, domain })`** — OAuth refresh for CRM API.
4. **Loop** every **120s** (2 minutes):
   - **`Promise.all(cycles.map(cycle => cycle.run(database, zohoClient)))`** — Each cycle knows a **Zoho module** and **table name**; pulls remote records and upserts locally (or equivalent—see each cycle file).
   - Per-cycle errors logged; **Sentry** can capture global failures.

**Features controlled:** Periodic **CRM ↔ Postgres** synchronization without blocking web traffic.

## Deployment

Root `package.json` — `deploy:worker-zoho-sync`, `deploy:worker-zoho-sync-staging`.
