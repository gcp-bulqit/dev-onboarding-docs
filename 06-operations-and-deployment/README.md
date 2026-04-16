# 6. Operations and deployment

How each component deploys, how to verify it worked, and how to
roll back if it didn't.

## Documents in this folder

| Doc | Topics |
|-----|--------|
| [deployment-workflow.md](./deployment-workflow.md) | End-to-end lifecycle: branch model, PR gates, merge triggers, CMS migrations, changeset versioning, staging vs production |
| [vercel-web.md](./vercel-web.md) | Web app on Vercel: auto-deploy, preview URLs, health checks, rollback, env vars, CI pipelines |
| [fly-io-workers.md](./fly-io-workers.md) | Workers on Fly.io: deploy triggers, manual deploy, verify, rollback, machine config, env vars per worker |

Read `deployment-workflow.md` first for the big picture. The other
two cover per-component mechanics.

## Quick reference

| Component | Production trigger | Staging trigger | Verify |
|-----------|-------------------|-----------------|--------|
| Web app | Push to `main` (auto) | Push to `staging`/PR (auto) | `curl https://bulqit.com/api/health` |
| Temporal workers | Push to `main` (path-filtered) | Push to `staging` (path-filtered) | `fly status -a bul-temporal-workers` |
| Zoho sync | Push to `main` (path-filtered) | Push to `staging` (path-filtered) | `fly status -a worker-zoho-sync` |
| Notifications | Push to `main` (path-filtered) | Push to `staging` (path-filtered) | `fly status -a worker-onboarding-notifications` |

## Infrastructure provisioning

Initial setup and disaster recovery provisioning lives in
a separate repo (`Bulqit/bul-infra`). See its
`runbooks/reconstruct-site.md` for full site reconstruction.
