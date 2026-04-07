# External review — project overview

This folder holds **onboarding and developer documentation** for new contributors. This document is **step 1**: a high-level table of contents of **features and surface areas** in the current Bulqit website monorepo. **Detailed topic docs** live in **numbered subfolders** (one per major section below).

Component-level UI documentation is maintained separately in [STORYBOOK.md](./STORYBOOK.md).

A video overview and walkthrough of the major sections is maintained in [VIDEO_OVERVIEW.md](./VIDEO_OVERVIEW.md)

---

## Table of contents

### 1. Repository & tooling → [01-repository-and-tooling/](./01-repository-and-tooling/README.md)

- **Monorepo** — [monorepo-and-workspaces.md](./01-repository-and-tooling/monorepo-and-workspaces.md)
- **Apps** — [apps-overview.md](./01-repository-and-tooling/apps-overview.md)
- **Shared packages** — [shared-packages-overview.md](./01-repository-and-tooling/shared-packages-overview.md)
- **Quality** — [quality-and-testing.md](./01-repository-and-tooling/quality-and-testing.md)
- **Releases** — [releases-and-changesets.md](./01-repository-and-tooling/releases-and-changesets.md)
- **Git hooks** — [git-hooks.md](./01-repository-and-tooling/git-hooks.md)

### 2. Web application (`apps/web` — Next.js) → [02-web-application/](./02-web-application/README.md)

- **Public marketing & content** — [public-marketing-and-cms.md](./02-web-application/public-marketing-and-cms.md)
- **Sanity Studio** — [sanity-studio.md](./02-web-application/sanity-studio.md)
- **Authentication & access** — [authentication-and-access.md](./02-web-application/authentication-and-access.md)
- **Account selection & registration** — [registration-and-onboarding.md](./02-web-application/registration-and-onboarding.md)
- **Member dashboard** — [member-dashboard.md](./02-web-application/member-dashboard.md)
- **Vendor dashboard** — [vendor-dashboard.md](./02-web-application/vendor-dashboard.md)
- **Admin console** — [admin-console.md](./02-web-application/admin-console.md)
- **Payments** — [payments.md](./02-web-application/payments.md)
- **API routes (`app/api`)** — [api-routes.md](./02-web-application/api-routes.md)

### 3. Background workers & automation → [03-background-workers-and-automation/](./03-background-workers-and-automation/README.md)

- **Temporal workers** — [temporal-workers.md](./03-background-workers-and-automation/temporal-workers.md)
- **Notification worker** — [notification-worker.md](./03-background-workers-and-automation/notification-worker.md)
- **Zoho sync worker** — [zoho-sync-worker.md](./03-background-workers-and-automation/zoho-sync-worker.md)

### 4. Shared packages (feature-oriented) → [04-shared-packages/](./04-shared-packages/README.md)

- **`packages/database`** — [database.md](./04-shared-packages/database.md)
- **`packages/sanity-cms`** — [sanity-cms.md](./04-shared-packages/sanity-cms.md)
- **`packages/notifications`** — [notifications.md](./04-shared-packages/notifications.md)
- **`packages/messaging`** — [messaging.md](./04-shared-packages/messaging.md)
- **`packages/temporal-workflows`** — [temporal-workflows.md](./04-shared-packages/temporal-workflows.md)
- **`packages/zoho-client`** — [zoho-client.md](./04-shared-packages/zoho-client.md)
- **`packages/zoho-support-client`** — [zoho-support-client.md](./04-shared-packages/zoho-support-client.md)
- **`packages/referral-factory-client`** — [referral-factory-client.md](./04-shared-packages/referral-factory-client.md)
- **`packages/subscription-billing`** — [subscription-billing.md](./04-shared-packages/subscription-billing.md)
- **`packages/vendors-bulqit-score`** — [vendors-bulqit-score.md](./04-shared-packages/vendors-bulqit-score.md)
- **`packages/display-ids`** — [display-ids.md](./04-shared-packages/display-ids.md)
- **`packages/utilities`** — [utilities.md](./04-shared-packages/utilities.md)
- **`packages/eslint-config`** — [eslint-config.md](./04-shared-packages/eslint-config.md)

### 5. Integrations (cross-cutting) → [05-integrations/](./05-integrations/README.md)

- **Clerk** — [clerk.md](./05-integrations/clerk.md)
- **Sanity** — [sanity.md](./05-integrations/sanity.md)
- **PostgreSQL** — [postgresql-drizzle.md](./05-integrations/postgresql-drizzle.md)
- **Temporal Cloud** — [temporal-cloud.md](./05-integrations/temporal-cloud.md)
- **Twilio** — [twilio.md](./05-integrations/twilio.md)
- **SendGrid** — [sendgrid.md](./05-integrations/sendgrid.md)
- **Stripe** — [stripe.md](./05-integrations/stripe.md)
- **Zoho CRM** — [zoho-crm.md](./05-integrations/zoho-crm.md)
- **Referral Factory** — [referral-factory.md](./05-integrations/referral-factory.md)
- **Segment / FullStory / Sentry / Userback** — [analytics-and-observability.md](./05-integrations/analytics-and-observability.md)

### 6. Operations & deployment → [06-operations-and-deployment/](./06-operations-and-deployment/README.md)

- **Vercel** — [vercel-web.md](./06-operations-and-deployment/vercel-web.md)
- **Fly.io** — [fly-io-workers.md](./06-operations-and-deployment/fly-io-workers.md)

### 7. Agent review (Cursor & Claude skills snapshot) → [07-agent-review/](./07-agent-review/README.md)

- **Agent config** — Root `AGENTS.md` / `CLAUDE.md`, `.claude/skills/gitnexus/*`, and consolidated **`.cursor/`** (`rules`, copies of agent files + skills); see [07-agent-review/README.md](./07-agent-review/README.md).

---

## Where to go next

- **Root `README.md`** — Install, scripts, env vars, Turbo, migrations overview.
- **`AGENTS.md`** — Agent-oriented commands, architecture notes, DB migration rules, testing safety (integration tests wipe DB).
- **`docs/`** — Additional guides (e.g. CMS migrations, Clerk setup).

Subsequent onboarding docs in this folder will link from this outline into setup, architecture, and team workflows.
