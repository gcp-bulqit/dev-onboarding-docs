# Notification worker

## Paths

| Item | Path |
|------|------|
| Entry | `apps/worker-notifications/src/index.ts` |
| Mail adapter | `apps/worker-notifications/src/services/mail.ts` (SendGrid) |
| SMS adapter | `apps/worker-notifications/src/services/sms.ts` (Twilio) |
| Environment | `apps/worker-notifications/src/environment.ts` |
| Package domain | `packages/notifications/` — `claimDueQueueItems`, `deliverQueueItem`, types |

## Behavior synopsis — `index.ts`

1. **`Sentry.init`** if DSN present.
2. **`setNotificationLogger`** from `@bulqit/notifications` with the worker logger.
3. **`getInstance`** from `@bulqit/database/instances/node` — pooled DB.
4. Builds **`DeliveryServices`**: site URL, optional template storage URL, **SendGrid** (`mail`) with from/reply/BCC/ASM config, **Twilio SMS** (`sms`).
5. **Infinite loop** every **5s**:
   - **`claimDueQueueItems(batchSize, database)`** — Takes up to 100 due rows.
   - **`Promise.all`** + **`deliverQueueItem`** per row — Renders templates, sends email/SMS, updates queue state.
   - Errors per item → **Sentry** with tags (`queueItemId`, `eventId`, `templateId`).

**Features controlled:** Reliable delivery of the **notification queue** (transactional email + SMS) decoupled from web requests.

## Deployment

Root `package.json` — `deploy:worker-notifications`, `deploy:worker-notifications-staging`, `deploy:worker-notifications-development`.
