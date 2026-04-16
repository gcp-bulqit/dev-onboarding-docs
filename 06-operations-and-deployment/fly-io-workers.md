# Fly.io (workers)

## Paths

| Worker | Directory | Process entry | Fly app (prod) | Fly app (staging) |
|--------|-----------|---------------|----------------|-------------------|
| Temporal | `apps/temporal-workers/` | `src/worker.ts` | `bul-temporal-workers` | `bul-temporal-workers-staging` |
| Notifications | `apps/worker-onboarding-notifications/` | `src/index.ts` | `worker-onboarding-notifications` | `worker-onboarding-notifications-staging` |
| Zoho sync | `apps/worker-zoho-sync/` | `src/index.ts` | `worker-zoho-sync` | `worker-zoho-sync-staging` |

Each includes `fly.toml` + `Dockerfile` for container deploy.

## How deploys work

Workers deploy via GitHub Actions on push, filtered by path.

| Worker | Production trigger | Staging trigger | Path filter |
|--------|-------------------|-----------------|-------------|
| Temporal | Push to `main` | Push to `staging` | `apps/temporal-workers/**`, `packages/temporal-workflows/**`, `pnpm-lock.yaml` |
| Notifications | Push to `main` | Push to `staging` | `apps/worker-onboarding-notifications/**`, `packages/**`, `pnpm-lock.yaml` |
| Zoho sync | Push to `main` | Push to `staging` | `apps/worker-zoho-sync/**`, `packages/**`, `pnpm-lock.yaml` |

All workflows also support manual trigger from the GitHub Actions
tab (workflow_dispatch).

### Deploy pipeline

Each workflow sets up pnpm + Node 22, then runs
`pnpm run deploy:worker-*`, which:

1. Runs `turbo prune` to extract the target package into `out/docker`
2. Runs `flyctl deploy` with the app-specific Fly app name

The Temporal workflow additionally pre-builds the workflows package
and worker before deploy. Zoho and notifications defer their build
to the Docker image step (`SKIP_PREPARE=true`).

## Manual deploy

From the `bul-website` repo root:

```bash
# Temporal
pnpm run deploy:worker-temporal              # production
pnpm run deploy:worker-temporal-staging      # staging

# Notifications
pnpm run deploy:worker-onboarding-notifications          # production
pnpm run deploy:worker-onboarding-notifications-staging   # staging

# Zoho sync
pnpm run deploy:worker-zoho-sync             # production
pnpm run deploy:worker-zoho-sync-staging     # staging
```

Requires `FLY_API_TOKEN` in the environment.

## Verify

```bash
fly status -a <app-name>     # machine state, deployment info
fly logs -a <app-name>       # tail live logs
```

What to look for:
- Machine state is `started` (not `stopped` or `failed`)
- Recent deployment timestamp matches what you just pushed
- Logs show the worker connecting and polling (no crash loops)

## Roll back

```bash
fly releases -a <app-name> --image                     # list releases with image refs
fly deploy --image <previous-image-ref> -a <app-name>  # redeploy old image
```

There is no one-click rollback. You redeploy a previous Docker
image. The `--image` flag on `fly releases` exposes the image
reference you pass to `fly deploy`.

## Machine configuration

| Worker | Region | auto_stop | min_machines | Memory | Restart |
|--------|--------|-----------|-------------|--------|---------|
| Temporal | `ord` (Chicago) | `stop` (can stop when idle) | 1 | 1 GB | default |
| Notifications | `lax` (Los Angeles) | `false` (always on) | 1 | 1 GB | `always` |
| Zoho sync | `lax` (Los Angeles) | `false` (always on) | 1 | 1 GB | `always` |

### Auto-stop behavior

The Temporal worker has `auto_stop_machines = 'stop'` with
`min_machines_running = 1`. Fly keeps one machine running in the
primary region (`ord`) at all times. Excess machines scaled up
under load may stop when idle and restart on demand.

Extra machines that restart pay a cold-start penalty on the next
inbound request. If that's a problem, change `auto_stop_machines`
to `false` in `apps/temporal-workers/fly.toml` and redeploy. The
tradeoff is every machine runs 24/7.

## Environment variables

Secrets are set via `fly secrets set` on each app.

### Temporal workers

| Variable | Where to find it |
|----------|-----------------|
| `TEMPORAL_API_KEY` | Temporal Cloud > API Keys |
| `TEMPORAL_ADDRESS` | Temporal Cloud > Namespace settings |
| `TEMPORAL_NAMESPACE` | Temporal Cloud > Namespace name |
| `TEMPORAL_SERVER_NAME_OVERRIDE` | TLS server name from Temporal Cloud namespace (required for cloud connections) |
| `TEMPORAL_TASK_QUEUE` | Matches the queue name workflows use (falls back to `default`) |

### Zoho sync

| Variable | Where to find it |
|----------|-----------------|
| `DB_URL` | Neon dashboard > Connection string (pooled) |
| `ZOHO_DOMAIN` | Zoho API domain for the account (e.g. `zohoapis.com`, `zohoapis.eu`) |
| `ZOHO_CRM_CLIENT_ID` | Zoho API Console > Client ID |
| `ZOHO_CRM_CLIENT_SECRET` | Zoho API Console > Client Secret |
| `ZOHO_CRM_REFRESH_TOKEN` | Zoho OAuth flow (long-lived) |
| `SENTRY_DSN` | Sentry > Project > Client Keys (optional) |

### Notifications (onboarding)

The current worker only reads the variables below. The SendGrid
sender, BCC, and ASM group are hardcoded in
`apps/worker-onboarding-notifications/src/services/mail.ts`.

| Variable | Where to find it |
|----------|-----------------|
| `DB_URL` | Neon dashboard > Connection string |
| `SITE_URL` | Production URL (e.g. `https://bulqit.com`) |
| `SITE_PAGE_HOW_IT_WORKS_URL` | Path or full URL for the "how it works" page |
| `SITE_PAGE_REFER_URL` | Path or full URL for the referral page |
| `SENDGRID_API_KEY` | SendGrid > Settings > API Keys |
| `SENTRY_DSN` | Sentry > Project > Client Keys (optional) |

The infra env template in `bul-infra/env/fly-notifications.env.example`
also lists Twilio, Clerk, Sanity, and additional SendGrid vars.
Those are not currently read by this worker. If that ever changes,
update this table and the template together.

### GitHub Actions secret

All deploy workflows need `FLY_API_TOKEN` set as a repository
secret on `Bulqit/bul-website`.
