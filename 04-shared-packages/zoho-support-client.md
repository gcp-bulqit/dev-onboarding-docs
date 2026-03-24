# `packages/zoho-support-client`

## Paths

| Item | Path |
|------|------|
| Client | `packages/zoho-support-client/src/services/client.ts` — exported as `ZohoClient` |
| Errors | `packages/zoho-support-client/src/services/zoho-error.ts` |
| Types | `packages/zoho-support-client/src/interfaces.ts` — ticket payloads, contacts, refs |

## Behavior synopsis

Separate from CRM: **`ZohoClient`** here targets **Zoho Desk / Support** APIs to **create or update support tickets** with structured payloads (`CreateZohoSupportTicketParams`, etc.).

**Features controlled:** Inbound email / support workflows that need a ticket record in Zoho Support (see support webhook routes under `apps/web/src/app/api/webhooks/`).
