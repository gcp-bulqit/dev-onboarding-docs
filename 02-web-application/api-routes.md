# API routes (`apps/web/src/app/api`)

Next.js **Route Handlers** (`route.ts`) implement REST-ish endpoints. Below: **paths + what the code typically does** (read each `route.ts` for full detail).

## Health and diagnostics

| Path | Behavior synopsis |
|------|-------------------|
| `api/health/route.ts` | **`GET`** — Runs **`checkServices()`** (`@/utils/healthCheck`), returns JSON with per-service status, overall healthy/unhealthy, HTTP **200 vs 503**. |
| `api/healthz/route.ts` | **`GET`** — Minimal JSON `{ ok, timestamp, version }` using **`VERCEL_GIT_COMMIT_SHA`** (or `unknown`). |
| `api/admin/db/diagnostics/route.ts` | Admin-only DB diagnostics (auth via admin helpers). |

## CMS

| Path | Behavior synopsis |
|------|-------------------|
| `api/draft-mode/enable/route.ts` | **`defineEnableDraftMode`** from `next-sanity/draft-mode` — enables **Sanity draft** cookies for preview. |

## Dashboard

| Path | Behavior synopsis |
|------|-------------------|
| `api/dashboard/alerts/*` | Read/mark **dashboard alerts** for authenticated dashboard users. |

## Member / vendor documents

| Path | Behavior synopsis |
|------|-------------------|
| `api/member/invoices/[id]/pdf/route.ts` | Streams or generates **invoice PDF** for a member invoice record. |
| `api/vendor/transactions/*/invoice/*` | Vendor **invoice PDF** and **Stripe** invoice helpers. |

## Admin batch / import / export

| Path pattern | Behavior synopsis |
|--------------|-------------------|
| `api/admin/export-*` | CSV/Excel exports (members, vendors, jobs, services, etc.) — query DB, stream file. |
| `api/admin/upload-*` | Bulk **upload** parsing and validation into DB. |
| `api/admin/payouts/run-batch/*` | Triggers or polls **batch payout** jobs. |

## Messaging (Twilio)

| Path pattern | Behavior synopsis |
|--------------|-------------------|
| `api/messaging/**` | CRUD for **Conversations**, participants, messages, SMS participants—delegates to **`MessagingServiceCore`** / DB. |
| `api/twilio/**` | Webhooks (**signature verification**), voice proxy callbacks. |
| `api/admin/messaging/**` | Admin tools: phone pool, support conversations, simulations. |

## Webhooks

| Path | Behavior synopsis |
|------|-------------------|
| `api/sendgrid/**` | **Inbound parse** / event webhooks for email pipeline. |
| `api/webhooks/support-inbound-email/**` | Routes inbound support email into ticketing/messaging. |

## Cron

| Path | Behavior synopsis |
|------|-------------------|
| `api/cron/*` | Vercel/cron invocations; typically checks **`CRON_SECRET`** header/query. Example: SMS pool health. |

## Examples / diagnostics

| Path | Behavior synopsis |
|------|-------------------|
| `api/sentry-example-api/route.ts` | Demonstrates Sentry capture on the server. |
| `api/temporal-example-api/route.ts` | Starts or queries a **Temporal** workflow for testing. |

**Cross-cutting:** Most handlers use **`NextResponse.json`**, Zod or manual validation, and either **Clerk session** (`auth()`) or **shared secret** (webhooks, cron). **Admin** routes often wrap **`withApiAdminAuth`** or equivalent—see each file’s imports.
