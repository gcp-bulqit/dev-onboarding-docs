# Development environment checklist

Use this list to stand up a **working local (or team-shared) development environment** for the Bulqit website monorepo. It is grounded in [infrastructure](./infra-overview.md), [repository tooling](./01-repository-and-tooling/README.md), [integrations](./05-integrations/README.md), and [operations](./06-operations-and-deployment/README.md). Commands and env var names live in the **application repository** root **`README.md`** and **`AGENTS.md`**—this doc only points there where details would go stale.

## How to use this list

- Check off **`- [ ]`** items as you complete them.
- You can run **only the web app** first; background workers and Temporal are optional until you need notifications, workflows, or CRM sync locally.
- Use a **disposable** database for **integration tests**—never point `DB_URL` at shared staging or production. See **`AGENTS.md`** in the code repo.
- For **full production-style reconstruction** (mise, Terraform, many services), see the [reconstruction flow](./infra-overview.md#reconstruction-flow) in the infrastructure overview; day-to-day dev usually does not require every step there.

---

## Prerequisites

| Done  | Step                                         | Notes                                                                                                | Doc                                                                                                                          |
| ----- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| - [ ] | **Node.js** installed                        | Match the version pinned in **`.nvmrc`** in the application repo.                                    | [Monorepo and workspaces](./01-repository-and-tooling/monorepo-and-workspaces.md)                                            |
| - [ ] | **pnpm** installed                           | Workspace uses `packageManager` in root `package.json`.                                              | Same; application **root `README.md`**                                                                                       |
| - [ ] | **Git** and clone access                     | Clone the **`bul-website`** mirror (or your team’s fork) per your GitHub access.                     | [Deployment model](./infra-overview.md#deployment-model)                                                                     |
| - [ ] | Optional: **CLI tools** for deploy debugging | **`flyctl`**, **Vercel CLI**, etc., only if you deploy or inspect those platforms from your machine. | [Fly.io (workers)](./06-operations-and-deployment/fly-io-workers.md), [Vercel](./06-operations-and-deployment/vercel-web.md) |

---

## Install dependencies

| Done | Step | Notes | Doc |
|------|------|-------|-----|
| - [ ] | **`pnpm install`** at monorepo root | Links all `apps/*`, `packages/*`, and `e2e-cypress` workspaces. | [Monorepo and workspaces](./01-repository-and-tooling/monorepo-and-workspaces.md) |
| - [ ] | Confirm **Turbo** runs | `pnpm build` / `pnpm dev` use `turbo run` from root `package.json`. | Same; [Turbo build context](./infra-overview.md#turbo-build-schema) |

---

## Configuration and secrets

| Done | Step | Notes | Doc |
|------|------|-------|-----|
| - [ ] | Create **`.env`** / **`.env.local`** files | Start from **`.env.local.example`** (or team template) in the application repo. Includes **`DB_URL`**, **`NEXT_PUBLIC_*`**, Clerk, Stripe, Twilio, Sanity, Temporal, etc., as needed for what you are testing. | Application **root `README.md`** (environment variables); [service inventory](./infra-overview.md) |
| - [ ] | **Clerk** keys for local sign-in | Required for dashboard and admin routes; middleware behavior is documented in the auth doc. | [Clerk](./05-integrations/clerk.md), [Authentication & access](./02-web-application/authentication-and-access.md) |
| - [ ] | **Sanity** project and tokens | Read token and public IDs for CMS pages and Studio. | [Sanity](./05-integrations/sanity.md), [Sanity Studio](./02-web-application/sanity-studio.md) |
| - [ ] | Optional: other integrations | Stripe, Twilio, SendGrid, Zoho, Referral Factory, Segment, Sentry, etc.—only if you exercise those code paths locally. | [Integrations index](./05-integrations/README.md) |

---

## Database (PostgreSQL / Drizzle)

| Done | Step | Notes | Doc |
|------|------|-------|-----|
| - [ ] | **Connection string** available | Typically **Neon** or a local Postgres; **`DB_URL`** / **`DATABASE_URL`** as used by `packages/database`. | [PostgreSQL (Drizzle)](./05-integrations/postgresql-drizzle.md) |
| - [ ] | **Migrations applied** | From repo root: `pnpm migrate` (Turbo **`migrate`** task) or follow **`AGENTS.md`** / database package scripts so schema matches code. | [PostgreSQL (Drizzle)](./05-integrations/postgresql-drizzle.md); **`AGENTS.md`** in application repo |
| - [ ] | Understand **integration tests** risk | Vitest integration tests may **truncate** tables—use a **throwaway** DB only. | [Quality and testing](./01-repository-and-tooling/quality-and-testing.md); **`AGENTS.md`** |

---

## CMS types and codegen

| Done | Step | Notes | Doc |
|------|------|-------|-----|
| - [ ] | Run **`pnpm typegen`** (monorepo root) | Regenerates Sanity-related TypeScript types via **`@bulqit/sanity-cms`**. | [Monorepo and workspaces](./01-repository-and-tooling/monorepo-and-workspaces.md); [`sanity-cms` package](./04-shared-packages/sanity-cms.md) |

---

## Run the Next.js web app

| Done | Step | Notes | Doc |
|------|------|-------|-----|
| - [ ] | **`pnpm dev`** or **`pnpm dev --filter=bulqit-website`** | Starts the **`bulqit-website`** app; filter avoids unrelated dev processes. | [Monorepo and workspaces](./01-repository-and-tooling/monorepo-and-workspaces.md); [Apps overview](./01-repository-and-tooling/apps-overview.md) |
| - [ ] | Open the local URL | Default Next.js port unless configured otherwise (see application **`package.json`**). | [Vercel](./06-operations-and-deployment/vercel-web.md) (production parity is Vercel-hosted) |

---

## Optional: background workers and Temporal

Use when you need **notification delivery**, **Temporal workflows**, or **Zoho sync** against your dev database—not required for basic UI work.

| Done | Step | Notes | Doc |
|------|------|-------|-----|
| - [ ] | **Temporal** dev server or cloud | Worker connects to **`TEMPORAL_*`** env; local dev server is described in the **application** Getting Started (notifications section). | [Temporal Cloud](./05-integrations/temporal-cloud.md), [Temporal workers](./03-background-workers-and-automation/temporal-workers.md) |
| - [ ] | Run **`apps/temporal-workers`** | Registers workflows from **`@bulqit/temporal-workflows`**. | [Temporal workers](./03-background-workers-and-automation/temporal-workers.md) |
| - [ ] | Run **`apps/worker-notifications`** | Delivers queued notifications (SendGrid / Twilio) using **`@bulqit/notifications`**. | [Notification worker](./03-background-workers-and-automation/notification-worker.md) |
| - [ ] | Run **`apps/worker-zoho-sync`** | CRM sync cycles if Zoho credentials are configured. | [Zoho sync worker](./03-background-workers-and-automation/zoho-sync-worker.md) |

End-to-end notification delivery from the web app typically requires **the same `DB_URL`** as the web app plus the workers above; see the **application root `README.md`** (“Notifications … locally”) and **`packages/notifications`** docs in that repo for the authoritative sequence.

---

## Storybook (UI components)

| Done | Step | Notes | Doc |
|------|------|-------|-----|
| - [ ] | From **`apps/web`**, run **`pnpm storybook`** | Serves on **port 6006** (scripts also compile Storybook CSS). | [STORYBOOK.md](./STORYBOOK.md) |

---

## Quality checks (before pushing)

| Done | Step | Notes | Doc |
|------|------|-------|-----|
| - [ ] | **`pnpm lint`** | Turbo fans out ESLint via **`@bulqit/eslint-config`**. | [Quality and testing](./01-repository-and-tooling/quality-and-testing.md) |
| - [ ] | **`pnpm check-types`** | TypeScript across packages. | Same |
| - [ ] | **`pnpm test`** (or **`test:unit`** in **`apps/web`**) | Vitest; prefer unit tests for fast feedback. | Same |
| - [ ] | Optional: **`pnpm test:integration`** | Only against a **disposable** database. | Same; **`AGENTS.md`** |

---

## Optional: end-to-end (Cypress)

| Done | Step | Notes | Doc |
|------|------|-------|-----|
| - [ ] | Install and run **`e2e-cypress`** package | Specs listed in quality doc; **`CYPRESS_BASE_URL`** may point at local or deployed preview. | [Quality and testing](./01-repository-and-tooling/quality-and-testing.md) |

---

## Troubleshooting

| Done  | Step                                                             | Notes                                                                                 | Doc                                                                               |
| ----- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| - [ ] | Clear **Turbo** cache if builds behave oddly                     | **`pnpm turbo clean`** or remove **`.turbo/`**; see application **root `README.md`**. | [Monorepo and workspaces](./01-repository-and-tooling/monorepo-and-workspaces.md) |
| - [ ] | Re-run **`pnpm install`** after pulling large dependency changes | Post-merge hooks may also run **`pnpm build:packages`** (Lefthook).                   | [Git hooks](./01-repository-and-tooling/git-hooks.md)                             |

---
