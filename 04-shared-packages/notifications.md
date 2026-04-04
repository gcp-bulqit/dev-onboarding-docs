# `packages/notifications`

## Paths

| Item | Path |
|------|------|
| Package | `packages/notifications/src/` |
| Public API | `packages/notifications/src/index.ts` (barrel exports) |

## Source structure

| Directory | Purpose |
|-----------|---------|
| `events/` | Event catalog, event names, metadata, payload maps |
| `recipients/` | Recipient resolution (who gets notified for each event) |
| `delivery/` | Template resolution, delivery channels, URL token handling |
| `delivery/channels/` | Per-channel delivery: email, SMS, inbox, dashboard alert |
| `email-templates/` | Email template token definitions and rendering |
| `templates/` | Template management |
| `services/` | Event creation and processing services |
| `cancellation/` | Notification cancellation logic |
| `idempotency/` | Deduplication for event processing |
| `preview/` | Template preview rendering |
| `notification-categories/` | Category definitions |
| `utilities/` | Shared helpers and error classes |
| `tests/` | Test suite |

## Key functions and types

From `index.ts` and submodules:

- **Event catalog** — `EVENT_NAMES`, `EVENT_METADATA`, `getEventMetadata`, `isValidEventName`, `VALID_TARGETS_BY_EVENT` — canonical list of notification events and who can be targeted.
- **Event / subject / condition helpers** — `EVENT_TYPES`, `SUBJECT_TYPES`, `CONDITIONS`, validators for admin UI and enqueue logic.
- **Queue delivery** — `claimDueQueueItems`, `deliverQueueItem`, `QueueItem`, `DeliveryServices` — used by `apps/worker-notifications` to send deliveries and persist state.
- **Delivery channels** — email (SendGrid), SMS (Twilio), inbox (in-app), dashboard alert. Four channels, selected per-template.
- **Logger hook** — `setNotificationLogger` — lets workers attach pino/Sentry-friendly logging.

## Complexity

This package contains the largest files in the codebase. `email-templates/tokens.ts`, `recipients/resolution.ts`, `delivery/template-resolution.ts`, and `delivery/url-tokens.ts` are all substantial files. The related `packages/temporal-workflows/src/notifications/run-notification-cron-bucket.ts` is also large. These files grow as new notification event types are added.

## Queue model

The notification system uses a database-backed queue (`notifications_queue` table). The worker polls every 5 seconds, claims batches of 100 items, and delivers through the appropriate channel. Failed deliveries are retried automatically: `delivery-orchestration.ts` checks `shouldRetry` per variant, resets the queue item to `pending`, and advances `scheduledAt` with backoff (up to 3 attempts). There is no separate dead-letter queue; items that exhaust retries are marked as failed in the queue table.

## Temporal integration

The Temporal worker auto-starts `processNotificationEventsContinuousWorkflow` on boot, which polls for pending notification events every 5 seconds. It also creates cron schedules for notification bucket processing (EST timezone).

**Features controlled:** Admin-configurable notification templates, event emission from app code, and at-least-once delivery through the DB-backed queue across four channels (email, SMS, inbox, dashboard alert).
