# SendGrid

## Paths

| Item | Path |
|------|------|
| Worker delivery | `apps/worker-notifications/src/services/mail.ts` |
| Webhooks | `apps/web/src/app/api/sendgrid/` — events, inbound parse |
| Package | `packages/notifications` — templates rendered before send |

## Behavior synopsis

**Transaction email** for the notification system: the notification worker passes **SendGrid** config (`from`, ASM group, reply routing secrets) into **`deliverQueueItem`**. Webhooks acknowledge bounces/opens and can ingest inbound mail for support flows.

**Features controlled:** **Templated email** from the notification queue and **email-driven** automations.
