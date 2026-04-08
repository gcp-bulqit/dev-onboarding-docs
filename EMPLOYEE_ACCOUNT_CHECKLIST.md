# Employee account checklist

Onboarding checklist for **third-party and platform accounts** (dashboard logins, org membership, and CLI access) that support development and operations on the Bulqit website stack. Items are derived from the [service inventory](./infra-overview.md) and the [integrations](./05-integrations/README.md) and [operations](./06-operations-and-deployment/README.md) docs in this repository.

## How to use this list

- **Not every row applies to every role.** For example, payments, CRM, or production-only tools are often limited to specific teams.
- Prefer **staging** or **read-only** access first where your process allows; production credentials stay in **Vercel**, **Fly**, or other managed stores—**do not** share secrets in chat or email.
- This document does **not** define internal approval chains; follow your manager or IT for who can grant each access.
- When the [infrastructure overview](./infra-overview.md) or integration set changes, update this checklist in the same pull request.

---

## Source control and CI
- [ ] **GitHub** — access to the `**bul-website`** mirror 
	- Clone, branch, review, and follow the deployment model (mirror → Vercel / Fly).
	- [Deployment model](./infra-overview.md#deployment-model)

---

## Hosting and deploy

- [ ] **Vercel** — project access for `**bul-website`**
	- View builds, env vars (as permitted), previews, and deployment logs for the Next.js app.
	- [Vercel (web app)](./06-operations-and-deployment/vercel-web.md)

- [ ] **Fly.io** — org / app access for worker apps
	- Deploy or inspect **temporal-workers**, **worker-notifications**, **worker-zoho-sync** (production, staging, preview, or development as appropriate).
	- [Fly.io (workers)](./06-operations-and-deployment/fly-io-workers.md)

---

## Data and content

- [ ] **Neon** — PostgreSQL hosting
	- Inspect branches, connection details (non-secret routing), or backups per your role; aligns with app DB state.
	- [Infrastructure overview](./infra-overview.md), [PostgreSQL / Drizzle](./05-integrations/postgresql-drizzle.md)

- [ ] **Sanity** — CMS project
	- Edit content, manage datasets, or run Studio against the correct project/dataset.
	- [Sanity](./05-integrations/sanity.md), [Sanity Studio (embedded)](./02-web-application/sanity-studio.md)

---

## Auth and application users

- [ ] **Clerk** — dashboard
	- Debug sessions, **metadata roles**, impersonation, and user records used by middleware and layouts.
	- [Clerk](./05-integrations/clerk.md), [Authentication & access](./02-web-application/authentication-and-access.md)

- [ ] **Manual users** (in application repo)
	- Follow team procedures for creating or fixing users outside self-serve flows.
	- `**docs/clerk-manual-user-setup.md`** (see [Where to go next](./README.md#where-to-go-next) in this folder’s README).

---

## Payments

- [ ] **Stripe** — dashboard (including **Connect** where relevant)
	- Webhooks, payment methods, Connect onboarding, payouts—typically **restricted** to finance/engineering leads.
	- [Stripe](./05-integrations/stripe.md), [Payments](./02-web-application/payments.md)

---

## Communications

- [ ] **Twilio**
	- SMS, Conversations, Flex-related assets as used by messaging and workers.
	- [Twilio](./05-integrations/twilio.md)

- [ ] **SendGrid**
	- Email delivery, templates, and inbound webhooks for notifications.
	- [SendGrid](./05-integrations/sendgrid.md)

---

## Workflows and CRM

- [ ] **Temporal Cloud**
	- Namespaces, task queues, and workflow history for `**apps/temporal-workers`**.
	- [Temporal Cloud](./05-integrations/temporal-cloud.md), [Temporal workers](./03-background-workers-and-automation/temporal-workers.md)

- [ ] **Zoho CRM** (and **Zoho Support / Desk** if your work touches support)
	- CRM sync worker and support integrations.
	- [Zoho CRM](./05-integrations/zoho-crm.md), [zoho-support-client](./04-shared-packages/zoho-support-client.md)

- [ ] **Referral Factory**
	- Referral campaigns and public `**/refer`** flows.
	- [Referral Factory](./05-integrations/referral-factory.md)

---

## Analytics and observability

- [ ] **Segment** (Twilio Segment)
	- Page analytics and destinations; **BigQuery** may receive data via Segment configuration rather than direct app code.
	- [Analytics and observability](./05-integrations/analytics-and-observability.md)

- [ ] **Sentry**
	- Error tracking (server, edge, client) as configured in the app.
	- [Analytics and observability](./05-integrations/analytics-and-observability.md)

- [ ] **FullStory**
	- Session replay when enabled.
	- [Analytics and observability](./05-integrations/analytics-and-observability.md)

- [ ] **Userback**
	- In-app feedback widget.
	- [Analytics and observability](./05-integrations/analytics-and-observability.md)

- [ ] **Google Analytics**
	- If used for marketing/analytics (may be env-driven).
	- [Analytics and observability](./05-integrations/analytics-and-observability.md)

---

## Other integrations

- [ ] **Google Maps** — API console / keys
	- Location features; keys are provided via env (see application `**README.md`** for `GOOGLE_MAPS_API_KEY`).
	- [Infrastructure overview](./infra-overview.md) (service inventory)

- [ ] **ZeroBounce**
	- Email verification when enabled (`ZERO_BOUNCE_API_KEY` in application env documentation).
	- Application `**README.md`** (environment variables).

---

## Local tooling (optional)

- [ ] **Mise and HCP Terraform**
	- One-time or disaster-recovery reconstruction flows; authenticate CLIs your team uses.
	- [Reconstruction flow](./infra-overview.md#reconstruction-flow)

---

