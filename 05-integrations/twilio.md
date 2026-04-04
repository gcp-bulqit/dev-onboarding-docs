# Twilio

## SDK packages

| Package | Usage |
|---------|-------|
| `twilio` | Server-side SDK (`packages/messaging/`) |
| `@twilio/conversations` | Client-side Conversations JS SDK for in-app messaging |

## Key areas

- **Domain package** — `packages/messaging/` handles conversation lifecycle, SMS bridging, phone pool allocation, and vendor contact management. See [messaging.md](../04-shared-packages/messaging.md).
- **API routes** — `apps/web/src/app/api/messaging/` for conversation CRUD, participant management, and media. `apps/web/src/app/api/twilio/` for webhook handling and voice proxy.
- **Flex integration** — `apps/web/src/app/api/messaging/flex/` for Flex configuration and conversation bridging. A Flex plugin at `tools/flex-plugin/bulqit-escalation-join/` provides an escalation join button for support agents.
- **Voice proxy** — `apps/web/src/app/api/twilio/voice/` handles voice call proxy callbacks.
- **Front-end** — `TwilioConversationChat.tsx` renders the messaging UI. Dashboard layouts wire `useTwilioContext` and `useConversations` for inbox polling.
- **Webhooks** — `api/twilio/conversations/webhook/` verifies signatures and updates conversation state on inbound events.

**Features controlled:** In-app messaging (member, vendor, support), SMS bridging with proxy numbers, voice proxy, Flex support escalation, and admin messaging diagnostics.
