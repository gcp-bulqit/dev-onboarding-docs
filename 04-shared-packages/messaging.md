# `packages/messaging`

## Paths

| Item | Path |
|------|------|
| Package | `packages/messaging/src/` |
| Core service | `packages/messaging/src/messagingService.ts` — `MessagingServiceCore` |

## Exports (synopsis) — `index.ts`

- **`MessagingServiceCore`** — Orchestrates conversation lifecycle, messages, and DB persistence (injected **`MessagingDb`** + **`MessagingTwilioGateway`**).
- **`TwilioConversationsGateway`**, **`getTwilioClient`**, **`getTwilioConversationsServiceSid`** — Low-level Twilio REST for Conversations API.
- **`PhonePoolService`** — Allocates/releases **proxy numbers** for SMS bridging; health checks.
- **`VendorContactService`** — Maps vendors to dialable numbers for ops flows.
- **`getFlexInstanceUrl`, `requireEnv`** — Config helpers.

**Features controlled:** **In-app messaging** between homeowners, vendors, and support—threads, SMS participants, proxy numbers, and admin diagnostics (`apps/web/src/app/api/messaging/**`).
