# `packages/temporal-workflows`

## Paths

| Item | Path |
|------|------|
| Barrel | `packages/temporal-workflows/src/all_workflows.ts`, `all_activities.ts` (imported by worker) |
| Workflow modules | `packages/temporal-workflows/src/**/workflows/*.ts` |

## Registered workflows (from `all_workflows.ts`)

| Workflow | Purpose | Trigger |
|----------|---------|---------|
| `example` | Sample | Manual (API route) |
| `hello` | Sample | Not currently triggered |
| `activateServiceWorkflow` | Messaging provisioning | App code |
| `process-notification-events` | One-off notification batch processing | App code (manual counterpart to continuous workflow) |
| `process-notification-events-continuous` | Continuous notification polling | Auto-started on worker boot |
| `run-notification-bucket` | Scheduled notification batch delivery | Cron schedule (created on worker boot) |
| `payment-dunning` | Failed payment follow-up | Cron schedule (created on worker boot) |
| `subscription-billing` | Billing cycles | Cron schedule (created on worker boot) |
| `update-vendor-scores` | Vendor score recomputation | Cron schedule (created on worker boot) |
| `weekly-payouts` | Payout batch orchestration | Cron schedule (created on worker boot) |

See [temporal-workers.md](../03-background-workers-and-automation/temporal-workers.md) for boot-time initialization details.

## Behavior synopsis

Each workflow module exports workflow functions (orchestration) and activity functions (side effects). The Temporal worker registers all exports. Some workflows are triggered by application code, others run continuously or on cron schedules created during worker startup.

**Features controlled:** Long-running, retriable processes that survive restarts: billing, payouts, notification fan-out, vendor scoring, and messaging provisioning.
