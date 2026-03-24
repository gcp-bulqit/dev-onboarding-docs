# `packages/zoho-client`

## Paths

| Item | Path |
|------|------|
| Client | `packages/zoho-client/src/client.ts` — `ZohoClient` |
| Types | `packages/zoho-client/src/interfaces.ts` — `ZohoError`, parameters |

## Behavior synopsis

**`ZohoClient`** wraps Zoho CRM REST with **OAuth refresh-token** flow (`clientId`, `clientSecret`, `refreshToken`, `domain`). Callers use typed methods to **list/upsert** CRM entities (exact methods in `client.ts`).

**Consumers:** **`apps/worker-zoho-sync`** runs periodic sync cycles; occasional server actions may read CRM data.

**Features controlled:** Keeping CRM records aligned with Bulqit’s operational data.
