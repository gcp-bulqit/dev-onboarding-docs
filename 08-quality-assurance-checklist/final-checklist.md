# Staging pre-release technical checklist

Human verification on **staging** before major releases or handoff milestones.

## How to use


| Item                 | Detail                                                                                                                                                                                                                                                   |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **When**             | Before promoting a milestone to production or external handoff; re-run if staging is redeployed after sign-off.                                                                                                                                          |
| **Environment**      | Staging web app base URL: `________________` (fill per run).                                                                                                                                                                                             |
| **Pass convention**  | Each row is binary: enter `**Pass`** or `**Fail`** in the Pass column for this run.                                                                                                                                                                      |
| **Notes / ticket**   | For any **Fail**, record repro, owner, and ClickUp id (e.g. `CU-…`). Leave blank when Pass.                                                                                                                                                              |
| **Ownership**        | Release engineer (or delegate) completes the matrix and blocks release on any unresolved Fail.                                                                                                                                                           |
| **Staging database** | **Read-only** for diagnostics (`SELECT` only). Do not run migrations, bulk writes, or `pnpm test:integration` against staging—that suite **truncates all tables**. See [AGENTS.md](../AGENTS.md) (Staging Database Protection, Integration Test Safety). |
| **Scope**            | Rows are representative high-risk surfaces. Add rows from release notes for features touching new webhooks, auth, payments, or workers.                                                                                                                  |


## Maintenance

- Update this file when new webhooks, auth paths, payment states, or Fly workers ship.
- Prefer linking fixes in **Notes / ticket** rather than long prose.
- Per-ID **incoming endpoints** and **outbound HTTPS** call lists live in the [appendix below](#appendix-endpoints-and-outbound-https).

---

## 1. Authentication and authorization (Clerk)


| ID      | Check                                                                                                   | Pass | Notes / ticket |
| ------- | ------------------------------------------------------------------------------------------------------- | ---- | -------------- |
| AUTH-01 | Member can sign in and reach member dashboard; session persists across refresh                          |      |                |
| AUTH-02 | Vendor can sign in and reach vendor dashboard; session persists across refresh                          |      |                |
| AUTH-03 | Admin can sign in and reach admin area; session persists across refresh                                 |      |                |
| AUTH-04 | Unauthenticated user cannot access protected member/vendor/admin routes (redirect or error as designed) |      |                |
| AUTH-05 | Admin-only server action or page rejects non-admin user (403 / no-op as designed)                       |      |                |
| AUTH-06 | If impersonation is used on staging: banner/flow behaves correctly and exit impersonation works         |      |                |


---

## 2. Payments (Stripe)


| ID     | Check                                                                                                                                               | Pass | Notes / ticket |
| ------ | --------------------------------------------------------------------------------------------------------------------------------------------------- | ---- | -------------- |
| PAY-01 | Member checkout flow completes through payment method capture (charge timing per product: after job completion)                                     |      |                |
| PAY-02 | Saved payment method / customer linkage behaves as expected after checkout                                                                          |      |                |
| PAY-03 | Stripe webhook endpoint reachable on staging; `payment_intent.succeeded` updates records / recovery path                                            |      |                |
| PAY-04 | Stripe webhook: `payment_intent.payment_failed` / dunning path behaves as expected                                                                  |      |                |
| PAY-05 | Stripe webhook: invoice-based success/failure paths (`invoice.payment_succeeded` / `invoice.payment_failed`) if subscriptions/invoices are in scope |      |                |
| PAY-06 | Stripe webhook: `transfer.created`, `transfer.failed`, `transfer.reversed` reflected in payout/vendor state as designed                             |      |                |
| PAY-07 | Stripe webhook: dispute events (`charge.dispute.created` / `updated` / `closed`) handled or safely logged                                           |      |                |
| PAY-08 | Stripe webhook: `account.updated` with transfers capability active triggers bank-verified behavior when applicable                                  |      |                |
| PAY-09 | Admin transaction hold release (or equivalent) works for a test case when in milestone scope                                                        |      |                |


Implementation reference: `apps/web/src/app/api/webhooks/stripe/route.ts`, `apps/web/src/actions/member/checkout/processCheckout.ts`, `apps/web/src/actions/admin/payments/releaseTransactionHoldAction.ts`.

---

## 3. State transitions (product workflows)


| ID       | Check                                                                                                          | Pass | Notes / ticket |
| -------- | -------------------------------------------------------------------------------------------------------------- | ---- | -------------- |
| STATE-01 | Order/job lifecycle: checkout through vendor completion and member-visible status updates                      |      |                |
| STATE-02 | Payment capture or billing step aligns with job completion rules for the milestone                             |      |                |
| STATE-03 | Vendor payout batch or disbursement state matches expectations after a completed test job (if in scope)        |      |                |
| STATE-04 | Service opportunity flow critical to release (create → assign → close or equivalent)                           |      |                |
| STATE-05 | Messaging conversation lifecycle critical to release (open → message → close/sync if applicable)               |      |                |
| STATE-06 | Temporal-driven workflow (if this release ships workflow changes): run completes or fails visibly in worker/UI |      |                |


References: `apps/temporal-workers`, `packages/temporal-workflows`.

---

## 4. APIs and integrations


| ID     | Check                                                                                                                        | Pass | Notes / ticket |
| ------ | ---------------------------------------------------------------------------------------------------------------------------- | ---- | -------------- |
| API-01 | `GET /api/health` returns 200; `database` and `sanity` services healthy in JSON                                              |      |                |
| API-02 | `GET /api/healthz` returns 200; `version` matches expected deploy commit when verified                                       |      |                |
| API-03 | Twilio conversations webhook: staging URL configured; test message or Twilio test event processed without 5xx                |      |                |
| API-04 | SendGrid inbound webhook: staging configured; inbound path returns expected status                                           |      |                |
| API-05 | Support inbound email webhook behaves as expected for a test message (if enabled on staging)                                 |      |                |
| API-06 | `POST /api/notifications/events` (or ops path in use) accepts authorized request and enqueues as designed                    |      |                |
| API-07 | Sanity CMS: staging dataset content loads; `POST /api/sanity/revalidate` succeeds with valid secret when testing CMS publish |      |                |
| API-08 | Smoke: any API paths called out in **this release’s** changelog return 2xx for happy path                                    |      |                |


Routes: `apps/web/src/app/api/health/route.ts`, `healthz`, `webhooks/stripe`, `twilio/conversations/webhook`, `sendgrid/inbound/webhook`, `webhooks/support-inbound-email`, `notifications/events`, `sanity/revalidate`. CMS pipeline: [cms-migrations.md](./cms-migrations.md).

---

## 5. Security


| ID     | Check                                                                                               | Pass | Notes / ticket |
| ------ | --------------------------------------------------------------------------------------------------- | ---- | -------------- |
| SEC-01 | Stripe webhook rejects requests without valid `stripe-signature` (400/4xx, not processed)           |      |                |
| SEC-02 | Sample admin API route returns 401/403 when unauthenticated (e.g. `apps/web/src/app/api/admin/...`) |      |                |
| SEC-03 | Rate limit or abuse-control path behaves consistently with staging config (if enabled)              |      |                |
| SEC-04 | No secrets or PII in client-visible responses for spot-checked endpoints                            |      |                |


---

## 6. Notifications


| ID       | Check                                                                                             | Pass | Notes / ticket |
| -------- | ------------------------------------------------------------------------------------------------- | ---- | -------------- |
| NOTIF-01 | Notification worker deployed on staging (`worker-notifications-staging` on Fly); process healthy  |      |                |
| NOTIF-02 | Representative email notification delivers to test inbox (SendGrid path)                          |      |                |
| NOTIF-03 | Representative SMS notification delivers or simulates correctly (Twilio path), if in scope        |      |                |
| NOTIF-04 | Duplicate or replay behavior acceptable for a critical template (idempotency / delay if required) |      |                |


Reference: `apps/worker-notifications/README.md`, `packages/notifications`.

---

## 7. Infrastructure and operations


| ID       | Check                                                                                                               | Pass | Notes / ticket |
| -------- | ------------------------------------------------------------------------------------------------------------------- | ---- | -------------- |
| INFRA-01 | Staging web deployment matches expected Git revision (Vercel/hosting dashboard vs `healthz`)                        |      |                |
| INFRA-02 | Fly worker: notification worker staging deploy current; logs without error storm                                    |      |                |
| INFRA-03 | Fly worker: Zoho sync staging deploy current; logs without error storm                                              |      |                |
| INFRA-04 | Fly worker: Temporal worker staging deploy current; logs without error storm                                        |      |                |
| INFRA-05 | Scheduled/cron routes used in staging executed or manually triggered successfully (`apps/web/src/app/api/cron/...`) |      |                |
| INFRA-06 | Error monitoring (e.g. Sentry): no new critical regressions spike after deploy                                      |      |                |


Workflows: `.github/workflows/deploy-worker-notifications-staging.yml`, `deploy-worker-zoho-sync-staging.yml`, `deploy-worker-temporal-staging.yml`.

---

## Appendix — Endpoints and outbound HTTPS

Use `{base}` as the staging web origin (e.g. `https://staging.example.com`). **Server Actions** are invoked as `POST` to the page URL that hosts the action, with Next.js `Next-Action` / `next-action` headers (see Network tab); they are not separate `/api/...` paths unless noted.

### AUTH-01

**Endpoints to check (incoming):**

- `GET {base}/dashboard/sign-in` — member sign-in entry.
- `GET {base}/dashboard/member` and at least one nested member route, e.g. `GET {base}/dashboard/member/services`, after auth.
- `POST {base}/dashboard/sign-in` or Clerk-hosted sign-in flow (browser redirect to Clerk, then back to app).

**Outbound HTTPS (during those requests):**

- **Clerk** — Backend API via `@clerk/nextjs/server` (`auth()`, and `clerkClient()` if `linkMemberByEmail` / `linkMemberByPhone` runs in `[member.ts](../apps/web/src/utils/member.ts)`).
- **Browser only** — Clerk session/JS from Clerk’s hosted frontend FAPI (Clerk account domain), not your Next route.

---

### AUTH-02

**Endpoints to check (incoming):**

- `GET {base}/sign-in` or vendor sign-in path your app uses.
- `GET {base}/dashboard/vendor`, `GET {base}/dashboard/vendor/my-jobs`, `GET {base}/dashboard/vendor/calendar` (or other vendor shell).

**Outbound HTTPS:**

- **Clerk** — `auth()` / `clerkClient()` as above for vendor session.

---

### AUTH-03

**Endpoints to check (incoming):**

- `GET {base}/admin/sign-in`
- `GET {base}/admin` (or other admin home after login).

**Outbound HTTPS:**

- **Clerk** — `auth()`; `validateAdminUser()` may imply additional Clerk/DB resolution per `[security.ts](../apps/web/src/utils/security.ts)`.

---

### AUTH-04

**Endpoints to check (incoming):**

- `GET {base}/dashboard/member/...` (no session cookie).
- `GET {base}/dashboard/vendor/...` (no session).
- `GET {base}/admin/...` (no session).

**Outbound HTTPS:**

- Typically **none** for a cold unauthenticated navigation if middleware resolves locally; **Clerk** only if the stack revalidates session.

---

### AUTH-05

**Endpoints to check (incoming):**

- **Server Action (example):** `POST {base}/admin/payouts/[id]` or `POST {base}/admin/financials/payouts/[id]` while invoking `releaseTransactionHold` from the page (see `[releaseTransactionHold` usage](../apps/web/src/app/admin/payouts/[id]/page.tsx)).
- **API (example):** `GET {base}/api/admin/messaging/conversations` **without** session cookies — expect 401 JSON when wrapped with `withApiAdminAuth` ([route](../apps/web/src/app/api/admin/messaging/conversations/route.ts)).

**Outbound HTTPS:**

- **Clerk** — when `withAdminAuth` / `validateAdminUser` / `withApiAdminAuth` runs.
- **None** for the handler body of `releaseTransactionHold` beyond DB (no Stripe in that action).

---

### AUTH-06

**Endpoints to check (incoming):**

- Admin flow that creates an impersonation session (product-specific URL).
- `GET {base}/dashboard/...` while impersonating (banner from `[ImpersonationBanner](../apps/web/src/components/shared/ImpersonationBanner.tsx)`).
- **Server Action:** `exitImpersonation` — `POST` to the page that triggers it; then `GET {base}/admin/sign-in?message=impersonation-ended` after `signOut`.

**Outbound HTTPS:**

- **Clerk** — `signOut`, actor token / session APIs as used by Clerk SDK.

---

### PAY-01

**Endpoints to check (incoming):**

- `GET {base}/dashboard/member/checkout`, `GET {base}/dashboard/member/checkout/payment`, `GET {base}/dashboard/member/checkout/success`.
- **Server Action:** `processCheckout` — `POST` to the checkout payment page (or parent) that submits the action (`[processCheckout.ts](../apps/web/src/actions/member/checkout/processCheckout.ts)`).

**Outbound HTTPS:**

- **Stripe API** — `stripe.paymentMethods.retrieve`, `stripe.customers.create`, `stripe.paymentMethods.attach`, `stripe.paymentMethods.list`, optional `stripe.paymentMethods.detach` via `getStripeClient()` (`[lib/stripe/client](../apps/web/src/lib/stripe/client.ts)`).
- **Twilio Conversations** (if provisioning runs) — via `provisionHomeownerForActiveServices` → Twilio REST when `canProvisionTwilio()` is true (`[provisionHomeownerForActiveServices.ts](../apps/web/src/server/messaging/provisionHomeownerForActiveServices.ts)`).

---

### PAY-02

**Endpoints to check (incoming):**

- Same as PAY-01 for initial save; then `GET {base}/dashboard/member/settings/payments` (or repeat checkout) to confirm persisted method.

**Outbound HTTPS:**

- **Stripe API** — same Stripe calls as PAY-01 when loading/attaching methods.
- **Clerk** — if pages call `auth()` / `getCurrentMember` path that hits `clerkClient()`.

---

### PAY-03

**Endpoints to check (incoming):**

- `POST {base}/api/webhooks/stripe` — body = raw Stripe event JSON; header `Stripe-Signature` required (`[route.ts](../apps/web/src/app/api/webhooks/stripe/route.ts)`).

**Outbound HTTPS:**

- **Stripe API** — `stripe.webhooks.constructEvent` (local verify, uses secret only); on success path `handlePaymentSucceeded` may call `stripe.paymentIntents.retrieve` / `stripe.invoices.retrieve` inside `resolveInvoiceDetailsFromPaymentIntent`; may call `stripe.charges.retrieve` for dispute-related flows.
- **Internal** — `createEvent` writes DB only (no HTTP) unless downstream worker sends email/SMS later.

---

### PAY-04

**Endpoints to check (incoming):**

- `POST {base}/api/webhooks/stripe` with event type `payment_intent.payment_failed` (and optionally `invoice.payment_failed`).

**Outbound HTTPS:**

- **Stripe API** — same entrypoint; invoice branch uses `stripe.paymentIntents.retrieve` before `handlePaymentFailed`.

---

### PAY-05

**Endpoints to check (incoming):**

- `POST {base}/api/webhooks/stripe` with `invoice.payment_succeeded` / `invoice.payment_failed`.

**Outbound HTTPS:**

- **Stripe API** — `stripe.paymentIntents.retrieve` for the invoice’s PaymentIntent, then `handlePaymentSucceeded` / `handlePaymentFailed`.

---

### PAY-06

**Endpoints to check (incoming):**

- `POST {base}/api/webhooks/stripe` with `transfer.created`, `transfer.failed`, `transfer.reversed`.

**Outbound HTTPS:**

- **Stripe API** — webhook verification only on entry; handlers update DB (`handleTransferCreated` / `handleTransferFailed` / `handleTransferReversed`) — no extra Stripe HTTP unless code paths add retrieves (confirm in same `[route.ts](../apps/web/src/app/api/webhooks/stripe/route.ts)` for your branch).

---

### PAY-07

**Endpoints to check (incoming):**

- `POST {base}/api/webhooks/stripe` with `charge.dispute.created`, `charge.dispute.updated`, `charge.dispute.closed`.

**Outbound HTTPS:**

- **Stripe API** — `stripe.charges.retrieve` (used in dispute handlers per `route.ts` grep); webhook verify on ingress.

---

### PAY-08

**Endpoints to check (incoming):**

- `POST {base}/api/webhooks/stripe` with `account.updated` (Connect account; include `previous_attributes` for capability transition).

**Outbound HTTPS:**

- **Stripe API** — webhook verify; `handleBankAccountVerified` may enqueue notifications (DB → worker → SendGrid/Twilio asynchronously).

---

### PAY-09

**Endpoints to check (incoming):**

- **Server Action:** `POST {base}/admin/payouts/[payoutId]` or `POST {base}/admin/financials/payouts/[payoutId]` when releasing hold ([pages](../apps/web/src/app/admin/payouts/[id]/page.tsx), `[financials/payouts](../apps/web/src/app/admin/financials/payouts/[id]/page.tsx)`).

**Outbound HTTPS:**

- **Clerk** — `withAdminAuth` → `validateAdminUser`.
- **None** to Stripe in `releaseTransactionHoldAction` itself (DB updates only).

---

### STATE-01

**Endpoints to check (incoming):**

- PAY-01 checkout endpoints (create order/job).
- **Server Action:** `POST` from `GET {base}/dashboard/vendor/my-jobs`, `GET {base}/dashboard/vendor/calendar`, or vendor home when calling `completeJobAction` (`[completeJobAction.ts](../apps/web/src/actions/vendors/jobs/completeJobAction.ts)`).
- `GET {base}/dashboard/member/jobs` or `GET {base}/dashboard/member/orders` for member-visible state.

**Outbound HTTPS:**

- **Stripe** — during checkout (PAY-01).
- **Twilio** — if completion triggers messaging/notifications that hit Twilio from server actions (via `createEvent` → worker, not synchronous HTTP from action unless code adds it).

---

### STATE-02

**Endpoints to check (incoming):**

- Same vendor **Server Action** as STATE-01 (`completeJobAction`), which calls `triggerPaymentForCompletedServiceJob`.

**Outbound HTTPS:**

- **Stripe API** — `stripe.paymentIntents.create` (`[triggerServiceJobPayment.ts](../apps/web/src/lib/payments/triggerServiceJobPayment.ts)`).

---

### STATE-03

**Endpoints to check (incoming):**

- Admin/UI paths for payouts (e.g. `GET {base}/admin/payouts/[id]`, `GET {base}/admin/financials/payouts/[id]`).
- If batch automation is used: `POST {base}/api/admin/payouts/run-batch` (and `GET {base}/api/admin/payouts/run-batch/status` if applicable) — verify in `[api/admin/payouts](../apps/web/src/app/api/admin/payouts/)`.

**Outbound HTTPS:**

- Depends on handler implementation: many payout routes are DB-only; if a route creates Stripe transfers, add **Stripe API** for that code path.

---

### STATE-04

**Endpoints to check (incoming):**

- Admin API examples: `GET/POST {base}/api/admin/service-opportunities/...`, `POST {base}/api/admin/service-opportunities/[id]/vendor`, `POST {base}/api/admin/service-opportunities/[id]/messaging/provision`, `POST {base}/api/admin/service-opportunities/[id]/messaging/sync`, `POST {base}/api/admin/service-opportunities/[id]/messaging/close` (`[service-opportunities](../apps/web/src/app/api/admin/service-opportunities/)`).
- Matching **Server Actions** from admin UI pages (same pattern: `POST` to hosting page).

**Outbound HTTPS:**

- **Twilio Conversations** — when provisioning/sync routes call `TwilioConversationsGateway` / `MessagingService`.
- **Clerk** — if routes use `withApiAdminAuth`.

---

### STATE-05

**Endpoints to check (incoming):**

- `POST {base}/api/twilio/conversations/webhook` — Twilio → your app.
- Member/vendor messaging pages under `GET {base}/dashboard/member/inbox/...`, `GET {base}/dashboard/vendor/inbox/...`.
- Additional `GET/POST {base}/api/messaging/...` routes used by the milestone (see `[app/api/messaging](../apps/web/src/app/api/messaging/)`).

**Outbound HTTPS:**

- On webhook: **Twilio REST** via `MessagingService` / `TwilioConversationsGateway` (`[messagingService.ts](../apps/web/src/server/messaging/messagingService.ts)`).
- On member pages: **Twilio** token or conversation APIs as invoked by those routes.

---

### STATE-06

**Endpoints to check (incoming):**

- `POST {base}/api/temporal-example-api` and/or `GET {base}/api/temporal-example-api` (`[temporal-example-api/route.ts](../apps/web/src/app/api/temporal-example-api/route.ts)`) if used as smoke.
- Any product route that starts workflows via `[startExampleWorkflow](../apps/web/src/lib/temporal-client.ts)` (search call sites for your release).

**Outbound HTTPS:**

- **Temporal Cloud / self-hosted Temporal frontend** — gRPC over TLS from `startExampleWorkflow` (Temporal client in Next server) and from `apps/temporal-workers` (worker → Temporal).

---

### API-01

**Endpoints to check (incoming):**

- `GET {base}/api/health`

**Outbound HTTPS:**

- **Sanity Content Lake** — `sanityClient.fetch` in `[checkSanity()](../apps/web/src/utils/healthCheck.ts)` (HTTPS to Sanity API host for your project).
- **Database** — PostgreSQL (TLS to Neon/host per `DB_URL`), not “HTTPS” in the REST sense but an external TCP dependency.

---

### API-02

**Endpoints to check (incoming):**

- `GET {base}/api/healthz`

**Outbound HTTPS:**

- **None** in `[healthz/route.ts](../apps/web/src/app/api/healthz/route.ts)` (returns JSON from env only).

---

### API-03

**Endpoints to check (incoming):**

- `POST {base}/api/twilio/conversations/webhook` — `application/x-www-form-urlencoded` body from Twilio.

**Outbound HTTPS:**

- **Twilio** — `verifyTwilioWebhookRequest` validates signature locally; `MessagingService.handleTwilioConversationsWebhook` performs Twilio REST calls as needed (`[webhook/route.ts](../apps/web/src/app/api/twilio/conversations/webhook/route.ts)`).

---

### API-04

**Endpoints to check (incoming):**

- `POST {base}/api/sendgrid/inbound/webhook` — SendGrid Inbound Parse (multipart/form-data).

**Outbound HTTPS:**

- **Vercel Blob** — `put(...)` from `@vercel/blob` for attachments (`[sendgrid/inbound/webhook/route.ts](../apps/web/src/app/api/sendgrid/inbound/webhook/route.ts)`).
- **Twilio Conversations** — `TwilioConversationsGateway` for mirroring inbound email into conversations.

---

### API-05

**Endpoints to check (incoming):**

- `POST {base}/api/webhooks/support-inbound-email` — JSON; header `x-support-webhook-secret`.

**Outbound HTTPS:**

- **Twilio Conversations** — `TwilioConversationsGateway.createConversation`, `MessagingService.sendMessage`, etc. (`[support-inbound-email/route.ts](../apps/web/src/app/api/webhooks/support-inbound-email/route.ts)`).
- **Zoho Desk** — `createTicket`, `appendConversationLinkToDescription` (`[zoho-support](../apps/web/src/server/zoho-support)`).

---

### API-06

**Endpoints to check (incoming):**

- `POST {base}/api/notifications/events` — JSON body; `Authorization: Bearer <NOTIFICATION_EVENTS_API_TOKEN>`.

**Outbound HTTPS:**

- **None** from handler after auth — `createEvent` inserts into Postgres only (`[create-event.ts](../apps/web/src/actions/notifications/create-event.ts)`). Delivery is **async** via worker → SendGrid/Twilio.

---

### API-07

**Endpoints to check (incoming):**

- `GET {base}/` (or any page using cached Sanity content) to confirm dataset reachable.
- `POST {base}/api/sanity/revalidate` — JSON; Bearer or body `secret` / `x-sanity-webhook-secret` (`[sanity/revalidate/route.ts](../apps/web/src/app/api/sanity/revalidate/route.ts)`).

**Outbound HTTPS:**

- **Sanity** — on normal page render, Next server fetches from Sanity API; revalidate endpoint only calls `revalidateTag` (no Sanity HTTP in that handler).

---

### API-08

**Endpoints to check (incoming):**

- List each `{base}/api/...` path from the release changelog.

**Outbound HTTPS:**

- Per-route: inspect matching `route.ts` for `fetch`, Stripe, Twilio, Sanity, Zoho, Clerk, Blob, etc.

---

### SEC-01

**Endpoints to check (incoming):**

- `POST {base}/api/webhooks/stripe` — omit `Stripe-Signature` or send garbage body (negative tests).

**Outbound HTTPS:**

- **None** before rejection (fails at `constructEvent` or missing header); **no** Stripe API calls on invalid signature path.

---

### SEC-02

**Endpoints to check (incoming):**

- `GET {base}/api/admin/messaging/conversations` (no cookies / no Bearer) — expect 401.
- Swap in any `withApiAdminAuth` route under `[api/admin](../apps/web/src/app/api/admin/)`.

**Outbound HTTPS:**

- **Clerk** — session validation attempt only.

---

### SEC-03

**Endpoints to check (incoming):**

- `POST {base}/api/rate-limit` with JSON `{ "actionType": "..." }` (`[rate-limit/route.ts](../apps/web/src/app/api/rate-limit/route.ts)`).
- Burst `POST {base}/api/notifications/events` until `429`.

**Outbound HTTPS:**

- **None** — limiter is in-process `RateLimiterMemory` (`[rate-limiter/limiters.ts](../apps/web/src/lib/rate-limiter/limiters.ts)`).

---

### SEC-04

**Endpoints to check (incoming):**

- Spot-check responses from `GET {base}/api/health`, `GET {base}/api/healthz`, and one authenticated member/vendor JSON endpoint.

**Outbound HTTPS:**

- Whatever those routes already use (see API-01 / API-02).

---

### NOTIF-01

**Endpoints to check (incoming):**

- **Not a Next route** — Fly app `worker-notifications-staging` process health (Fly `/metrics` or `fly logs`, not `{base}`).

**Outbound HTTPS (worker process):**

- **SendGrid** — mail API via worker `mail` service.
- **Twilio** — SMS API via worker `sms` service.
- **PostgreSQL** — notification queue DB.
- **Sentry** — `sentry.io` when `SENTRY_DSN` set (`[worker-notifications/src/index.ts](../apps/worker-notifications/src/index.ts)`).

---

### NOTIF-02

**Endpoints to check (incoming):**

- Trigger via **Server Action** or `POST {base}/api/notifications/events` that enqueues an email template; worker drains queue.

**Outbound HTTPS:**

- **SendGrid** from worker (see NOTIF-01).

---

### NOTIF-03

**Endpoints to check (incoming):**

- Same as NOTIF-02 but for SMS template/event.

**Outbound HTTPS:**

- **Twilio** from worker.

---

### NOTIF-04

**Endpoints to check (incoming):**

- Double `POST {base}/api/notifications/events` with same logical event, or replay webhook; optionally duplicate product action that calls `createEvent`.

**Outbound HTTPS:**

- Same as NOTIF-02/03 depending on channel; verify duplicate policy in DB/worker only.

---

### INFRA-01

**Endpoints to check (incoming):**

- `GET {base}/api/healthz` — compare `version` to deployment SHA.

**Outbound HTTPS:**

- None in handler.

---

### INFRA-02 — INFRA-04

**Endpoints to check (incoming):**

- Fly.io dashboard / `fly status` / `fly logs` for each staging worker (not `{base}`).

**Outbound HTTPS (workers):**

- **INFRA-02:** SendGrid, Twilio, DB, Sentry (see NOTIF-01).
- **INFRA-03:** **Zoho CRM API** via `@bulqit/zoho-client` (`zohoClient.post`, etc. in `[worker-zoho-sync](../apps/worker-zoho-sync/src/services/cycle.ts)`).
- **INFRA-04:** Temporal frontend (TLS), plus any dependencies in `apps/temporal-workers`.

---

### INFRA-05

**Endpoints to check (incoming):**

- `GET {base}/api/cron/phone-pool-sync` — header `Authorization: Bearer <CRON_SECRET>` (`[phone-pool-sync/route.ts](../apps/web/src/app/api/cron/phone-pool-sync/route.ts)`).
- `GET {base}/api/cron/sms-pool-health` — same auth (`[sms-pool-health/route.ts](../apps/web/src/app/api/cron/sms-pool-health/route.ts)`).

**Outbound HTTPS:**

- **Twilio** — `getTwilioClient().incomingPhoneNumbers.list()` and number updates in phone-pool-sync.
- **Slack** (if configured) — `fetch(slackWebhookUrl)` from `[poolAlerts.ts](../apps/web/src/server/messaging/alerts/poolAlerts.ts)` when pool unhealthy on sms-pool-health.

---

### INFRA-06

**Endpoints to check (incoming):**

- **Sentry UI** (`https://sentry.io` or your host) — filter by staging environment/release.

**Outbound HTTPS:**

- From app/workers to Sentry ingest when errors occur (existing SDK behavior).

---

## Sign-off


| Role                       | Name | Date |
| -------------------------- | ---- | ---- |
| Ran checklist              |      |      |
| Approved release / handoff |      |      |


