# Temporal Cloud

## Paths

| Item | Path |
|------|------|
| Workflows | `packages/temporal-workflows/` |
| Worker process | `apps/temporal-workers/src/worker.ts` |
| Example API | `apps/web/src/app/api/temporal-example-api/` |

## Behavior synopsis

**Temporal** runs **workflows** (orchestration) and **activities** (side effects) with automatic retries and durable history. Env vars (`TEMPORAL_ADDRESS`, `TEMPORAL_NAMESPACE`, `TEMPORAL_TASK_QUEUE`, `TEMPORAL_API_KEY`) connect the worker and any client code in Vercel.

**Features controlled:** Reliable background jobs—billing, payouts, notification fan-out, vendor score recomputation—without implementing your own queue infrastructure for those flows.
