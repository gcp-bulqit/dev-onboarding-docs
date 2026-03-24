# Applications overview

## `apps/web` — Next.js site (`bulqit-website`)

| Item | Path | Role |
|------|------|------|
| App Router | `apps/web/src/app/` | Routes, layouts, `route.ts` API handlers, metadata. |
| UI | `apps/web/src/components/` | Feature and shared React components. |
| Server actions | `apps/web/src/actions/` | Mutations invoked from the UI. |
| Edge / Clerk | `apps/web/src/proxy.ts` | **Clerk middleware**: public-route bypass, `auth.protect()`, role gates, sign-in redirects, impersonation handling, `redirect_url` cookie (see [authentication doc](../02-web-application/authentication-and-access.md)). |
| Config | `apps/web/next.config.ts`, `apps/web/package.json` | Next bundling, images, scripts (`dev`, `build`, tests). |

**Behavior synopsis:** The web app is the primary user-facing surface: marketing pages, CMS pages, member/vendor dashboards, admin, REST APIs under `/api`, and embedded Sanity Studio. The **`proxy.ts`** default export is the Clerk middleware entry (with `config.matcher`) that secures dashboard/admin routes and steers users after login.

## `apps/temporal-workers`

| Item | Path | Role |
|------|------|------|
| Worker runtime | `apps/temporal-workers/src/worker.ts` | Connects to **Temporal Cloud**, registers **workflows** and **activities** from `@bulqit/temporal-workflows`, runs an **Express** server for `/health` and `/workflows` introspection. |
| Logger | `apps/temporal-workers/src/logger.ts` | Logging for the worker process. |
| Build | `apps/temporal-workers/tsup.config.ts` | Bundles the worker for deployment. |
| Deploy | `apps/temporal-workers/fly.toml`, `Dockerfile` | Fly.io container layout. |

**Behavior synopsis:** Polls/executes Temporal tasks on the configured task queue; exposes health and discovery endpoints for operators. Uses `setNotificationLogger` so notification-related activities share logging with `@bulqit/notifications`.

## `apps/worker-notifications`

| Item | Path | Role |
|------|------|------|
| Entry | `apps/worker-notifications/src/index.ts` | Infinite loop: `claimDueQueueItems` → `deliverQueueItem` from `@bulqit/notifications` with **SendGrid** + **Twilio SMS** adapters; Sentry on failures. |

**Behavior synopsis:** Background delivery of the notification queue stored in Postgres—email (SendGrid) and SMS (Twilio) according to template and event configuration.

## `apps/worker-zoho-sync`

| Item | Path | Role |
|------|------|------|
| Entry | `apps/worker-zoho-sync/src/index.ts` | On an interval, runs every **sync cycle** in `cycles`: each cycle calls Zoho APIs via `ZohoClient` and upserts into the DB. |

**Behavior synopsis:** Keeps selected CRM entities in sync between Zoho and the application database on a fixed timer.

## Related

- [02 — Web application](../02-web-application/) — UI routes and features.
- [03 — Workers](../03-background-workers-and-automation/) — deeper worker notes.
