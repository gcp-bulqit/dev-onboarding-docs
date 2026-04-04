# `packages/messaging`

## Paths

| Item | Path |
|------|------|
| Package | `packages/messaging/src/` |
| Core service | `packages/messaging/src/messagingService.ts` — `MessagingServiceCore` |
| Twilio gateway | `packages/messaging/src/twilio/` |
| API routes | `apps/web/src/app/api/messaging/` |
| Admin messaging routes | `apps/web/src/app/api/admin/messaging/` |
| Front-end chat | `apps/web/src/components/messaging/TwilioConversationChat.tsx` |
| Flex plugin | `tools/flex-plugin/bulqit-escalation-join/` |

## Exports — `index.ts`

- **`MessagingServiceCore`** — Orchestrates conversation lifecycle, messages, and DB persistence (injected `MessagingDb` + `MessagingTwilioGateway`).
- **`TwilioConversationsGateway`**, **`getTwilioClient`**, **`getTwilioConversationsServiceSid`** — Low-level Twilio REST for Conversations API.
- **`PhonePoolService`** — Allocates/releases proxy numbers for SMS bridging; health checks.
- **`VendorContactService`** — Maps vendors to dialable numbers for ops flows.
- **`getFlexInstanceUrl`, `requireEnv`** — Config helpers.

## Complexity

`messagingService.ts` and the user conversations API route (`api/messaging/user/conversations/route.ts`) are both large files. The front-end `TwilioConversationChat.tsx` is also substantial. These are the primary maintenance surface for the messaging domain.

The user conversations route bulk-loads conversation read status but runs per-conversation unread-count queries. This N+1 pattern in unread-count calculation is flagged in the architecture assessment.

## Flex plugin

A Twilio Flex plugin lives at `tools/flex-plugin/bulqit-escalation-join/`. It provides an escalation join button for support agents in the Flex UI.

**Features controlled:** In-app messaging between homeowners, vendors, and support — threads, SMS participants, proxy numbers, voice proxy, Flex integration, and admin diagnostics.
