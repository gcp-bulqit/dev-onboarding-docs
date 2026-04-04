# Temporal workers

## Paths

| Item | Path |
|------|------|
| Worker app | `apps/temporal-workers/src/worker.ts` |
| Logger | `apps/temporal-workers/src/logger.ts` |
| Bundle config | `apps/temporal-workers/tsup.config.ts` |
| Deploy | `apps/temporal-workers/fly.toml`, `Dockerfile` |
| Workflow definitions | `packages/temporal-workflows/` |

## Behavior synopsis — `worker.ts`

- **Express** app on **`HEALTH_CHECK_PORT`** (default **3001**):
  - **`GET /health`** — JSON `{ status: healthy, timestamp, service: temporal-worker }`.
  - **`GET /workflows`** — Lists exported **workflow** names and **activity** keys from `@bulqit/temporal-workflows/all_*` plus env (`TEMPORAL_TASK_QUEUE`, `TEMPORAL_NAMESPACE`, `TEMPORAL_ADDRESS`).
  - **`GET /`** — Discovery of endpoints.
- **`run()`** — Validates **`TEMPORAL_ADDRESS`** and **`TEMPORAL_NAMESPACE`**, starts **`NativeConnection`**, creates **`Worker`** with:
  - **`workflowsPath`** / **`activities`** from the compiled workflow package bundles.
  - Task queue from env.
- **`setNotificationLogger`** — Bridges logging to `@bulqit/notifications` for activities that emit notifications.
- **Worker** runs until process exit; connects to **Temporal Cloud** (or compatible server).

## Boot-time initialization

After creating the Worker instance but before it begins accepting tasks, `worker.ts` performs initialization:

- **Starts `processNotificationEventsContinuousWorkflow`** — a long-running workflow that polls for pending notification events every 5 seconds. Started as a singleton (idempotent).
- **Creates cron schedules** (idempotent, skips if already created):
  - Notification bucket schedules (EST timezone crons)
  - Vendor score recalculation
  - Payment dunning
  - Subscription billing
  - Weekly payouts

All schedule creation is wrapped in try/catch so an existing schedule does not prevent the worker from starting.

## Deployment

Root `package.json` — `deploy:worker-temporal*` scripts prune the monorepo and deploy this image to Fly.io.

**Features controlled:** Async business processes — notification event processing (continuous), notification bucket delivery (scheduled), subscription billing, payment dunning, weekly payouts, vendor score updates, messaging activation.
