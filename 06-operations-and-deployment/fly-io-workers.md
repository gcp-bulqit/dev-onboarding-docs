# Fly.io (workers)

## Paths

| Worker | Directory | Process entry |
|--------|-----------|----------------|
| Temporal | `apps/temporal-workers/` | `src/worker.ts` |
| Notifications | `apps/worker-notifications/` | `src/index.ts` |
| Zoho sync | `apps/worker-zoho-sync/` | `src/index.ts` |

Each includes **`fly.toml`** + **`Dockerfile`** for container deploy.

## Behavior synopsis

Root **`deploy:worker-*`** scripts:

1. **`turbo prune`** the target package into `out/docker`.
2. **`flyctl deploy`** with the app-specific **Fly** app name (production/staging/preview variants).

These processes are **long-running** (loops or Temporal poll) unlike Vercel functions—Fly keeps VMs alive and exposes internal health ports where configured.

**Features controlled:** **Background capacity**—workflows, notification delivery, CRM sync—outside request/response timeouts.
