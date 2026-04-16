# Vercel (web app)

## Paths

| Item | Path |
|------|------|
| App | `apps/web/` (package name `bulqit-website`) |
| Build orchestration | Root `turbo.json` |
| Health check (deep) | `apps/web/src/app/api/health/route.ts` |
| Health check (liveness) | `apps/web/src/app/api/healthz/route.ts` |

## How deploys work

Vercel auto-deploys on every push. No manual steps needed.

| Branch | Deploy type | URL |
|--------|------------|-----|
| `main` | Production | `bulqit.com` |
| `staging` | Preview | Auto-generated Vercel URL |
| Any PR branch | Preview | Auto-generated Vercel URL |

Vercel runs `turbo run build --filter=bulqit-website...` on push.
Turbo remote cache speeds rebuilds. Database migrations run as a
build dependency: `turbo.json` makes `bulqit-website#build` depend
on `@bulqit/database#migrate`, so migrations execute before the
web app builds.

## Preview deploys

Every PR gets a unique preview URL. Find it in the PR's deployment
status check on GitHub, or in the Vercel dashboard under
Deployments.

Preview URLs are protected by Vercel Authentication. To bypass
for CI or automated testing, append the query parameter
`?x-vercel-protection-bypass=<secret>` using the
`VERCEL_AUTOMATION_BYPASS_SECRET`.

The Cypress E2E workflow (`e2e-cypress.yml`) uses this to run
tests against preview deploys automatically.

## Manual deploy

From the `bul-website` repo:

```bash
vercel --prod        # production
vercel               # preview (current branch)
```

## Verify

Liveness check (always 200 if the server is up):

```bash
curl https://bulqit.com/api/healthz
```

Deep check (database + Sanity CMS, returns 200 or 503):

```bash
curl https://bulqit.com/api/health
```

Response includes per-service status (`database`, `sanity`),
environment, and version.

After deploy, also check:
- Vercel dashboard shows "Ready" status
- No new errors in Sentry

## Roll back

Vercel dashboard > Deployments > find the last good deploy >
three-dot menu > "Promote to Production". Takes effect instantly.

From the CLI:

```bash
vercel ls                         # list recent deployments
vercel promote <deployment-url>   # promote a specific deploy
```

## Environment variables

Most env vars are set via the Vercel project settings or pushed
from `bul-infra` using `mise run vercel:push-env`.

### Database URL selector

The web app picks its database URL based on environment. See
`packages/utilities/src/environment/getDatabaseUrl.ts`:

| Vercel environment | Reads from |
|-------------------|------------|
| Preview | `DATABASE_URL` (injected per-branch by Neon) |
| Production, Staging, Local | `DB_URL` |

`DB_URL` must be set on the Vercel project for production and
staging to connect to the database. The Neon integration manages
`DATABASE_URL` for preview branches.

See `bul-infra/env/vercel-production.env.example` for the full
list of required variables.

## Post-deploy validation workflows

Two workflows run against the Vercel preview after a deploy
completes. Vercel deploys happen independently of these; they
run on the produced deployment.

| Workflow | Trigger | What it does |
|----------|---------|-------------|
| `e2e-cypress.yml` | Push/PR to `main` or `staging` | Waits for the Vercel deploy, runs Cypress against the preview URL. Can fail the workflow. |
| `lighthouse-ci.yml` | PRs only | Runs Lighthouse on 5 pages, posts scores as a PR comment. Exits success even if scores drop. |

Lint, unit, and integration tests run on PRs independently of
Vercel. See [quality-and-testing.md](../01-repository-and-tooling/quality-and-testing.md).
