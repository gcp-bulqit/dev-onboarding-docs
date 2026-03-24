# Twilio

## Paths

| Item | Path |
|------|------|
| Domain package | `packages/messaging/` — Conversations, SMS, phone pool |
| API routes | `apps/web/src/app/api/messaging/`, `apps/web/src/app/api/twilio/` |
| Dashboards | `MemberDashboardLayout`, `VendorDashboardLayout` — `useTwilioContext`, `useConversations` |

## Behavior synopsis

- **Conversations API** — Threaded chat between parties with optional **SMS** participants.
- **Phone pool** — Allocates **proxy numbers** for SMS so personal numbers stay private.
- **Webhooks** — `api/twilio/**` verifies signatures and updates conversation state on inbound events.

**Features controlled:** **Inbox/messaging** UX for members and vendors, and **admin** messaging tooling.
