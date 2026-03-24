# `packages/temporal-workflows`

## Paths

| Item | Path |
|------|------|
| Barrel | `packages/temporal-workflows/src/all_workflows.ts`, `all_activities.ts` (imported by worker) |
| Workflow modules | `packages/temporal-workflows/src/**/workflows/*.ts` |

## Registered workflows (from `all_workflows.ts`)

Exports include (non-exhaustive):

- `example/workflow`, `hello/workflow` — Samples.
- **`messaging/.../activateServiceWorkflow`** — Messaging provisioning.
- **`notifications/.../process-notification-events`**, **`process-notification-events-continuous`**, **`run-notification-bucket`** — Notification pipelines.
- **`payment-dunning/.../payment-dunning`** — Failed payment follow-up.
- **`subscription-billing/.../subscription-billing`** — Billing cycles.
- **`vendor-scores/.../update-vendor-scores`** — Score recompute jobs.
- **`weekly-payouts/.../weekly-payouts`** — Payout batch orchestration.

## Behavior synopsis

Each file typically exports **`workflow`** functions (orchestration) and **`activity`** functions (DB/API side effects). The **Temporal worker** registers all exports; the **web app** may start workflows via `@temporalio/client` in APIs or admin tools.

**Features controlled:** Long-running, retriable processes that must survive process restarts—billing, payouts, heavy notification fan-out, vendor scoring, etc.
