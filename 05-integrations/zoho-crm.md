# Zoho CRM

## Paths

| Item | Path |
|------|------|
| Client | `packages/zoho-client/` — `ZohoClient` |
| Worker | `apps/worker-zoho-sync/src/index.ts` — periodic `cycle.run` |

## Behavior synopsis

OAuth **refresh token** grants API access without interactive login. Sync cycles pull/push CRM modules defined in **`cycles`** so sales/ops tools stay aligned with Bulqit records.

**Features controlled:** **CRM alignment** for leads/contacts/deals (per configured modules).
