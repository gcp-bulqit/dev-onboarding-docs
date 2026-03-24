# PostgreSQL (Drizzle)

## Paths

| Item | Path |
|------|------|
| Schemas & migrations | `packages/database/` |
| App usage | Server actions, `apps/web/src/server/`, API routes, workers via `@bulqit/database/instances/*` |

## Behavior synopsis

**Drizzle** maps TypeScript objects to SQL. Workers use **`instances/node`**; serverless routes use **`instances/serverless`** (or HTTP) to avoid connection exhaustion. All **business entities**—users, vendors, jobs, notification queue rows, messaging state—persist here.

**Features controlled:** Authoritative **application state** (not Sanity content).
