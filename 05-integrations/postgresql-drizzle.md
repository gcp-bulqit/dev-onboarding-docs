# PostgreSQL (Drizzle)

## Paths

| Item | Path |
|------|------|
| Schemas | `packages/database/src/schemas/` |
| Migrations | `packages/database/migrations/` |
| Connection instances | `packages/database/src/instances/` |

## Connection instances

| Instance | Usage |
|----------|-------|
| `serverless` | Vercel serverless/edge functions (uses `@neondatabase/serverless`) |
| `node` | Workers and Node.js processes (WebSocket via `ws`) |
| `http` | HTTP-only connections (no transactions or WebSockets) |
| `integrations` | Node.js driver compatibility wrapper |

## Behavior synopsis

Drizzle ORM maps TypeScript objects to SQL. The database is hosted on Neon (serverless PostgreSQL). Connection instances are chosen based on the runtime environment to avoid connection exhaustion in serverless contexts.

All business entities persist here: users, vendors, lots, blocks, services, jobs, payment transactions, notification queue, messaging state, and more.

## Migration workflow

Migrations are generated from schema changes, not written manually. See `AGENTS.md` in the code repo for the full workflow. Key rule: always use `pnpm drizzle generate` from `packages/database/`, never write migrations by hand (manual migrations break Drizzle's snapshot chain).

## Known gaps

- The `vendor_jobs` table does not exist in any migration.
- The user conversations API route (`api/messaging/user/conversations/`) has a per-conversation unread-count query pattern that does not scale efficiently with conversation count.

**Features controlled:** Authoritative application state (not Sanity content).
