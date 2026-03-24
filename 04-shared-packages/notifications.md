# `packages/notifications`

## Paths

| Item | Path |
|------|------|
| Package | `packages/notifications/src/` |
| Public API | `packages/notifications/src/index.ts` (barrel exports) |

## Key functions & types (synopsis)

From **`index.ts`** (and submodules):

- **Event catalog** — `EVENT_NAMES`, `EVENT_METADATA`, `getEventMetadata`, `isValidEventName`, `VALID_TARGETS_BY_EVENT` — canonical list of **notification events** and who can be targeted.
- **Event / subject / condition helpers** — `EVENT_TYPES`, `SUBJECT_TYPES`, `CONDITIONS`, validators for admin UI and enqueue logic.
- **Queue delivery** — `claimDueQueueItems`, `deliverQueueItem`, `QueueItem`, `DeliveryServices` — used by **`apps/worker-notifications`** to send **SendGrid** + **Twilio** deliveries and persist state.
- **Logger hook** — `setNotificationLogger` — lets workers attach **pino**/Sentry-friendly logging.

**Features controlled:** Admin-configurable **notification templates**, **event emission** from app code, and **at-least-once** delivery through the DB-backed queue.
