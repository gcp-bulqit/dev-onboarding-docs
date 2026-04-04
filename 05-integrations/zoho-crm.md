# Zoho CRM

## Paths

| Item | Path |
|------|------|
| CRM client | `packages/zoho-client/` — `ZohoClient` for Zoho CRM API |
| Support client | `packages/zoho-support-client/` — separate client for Zoho Desk/Support API |
| Sync worker | `apps/worker-zoho-sync/src/index.ts` — periodic `cycle.run` |
| Support webhook | `apps/web/src/app/api/webhooks/support-inbound-email/route.ts` |

## Behavior synopsis

**Zoho CRM** — OAuth refresh token grants API access without interactive login. The sync worker pulls/pushes CRM modules on a 2-minute interval so sales/ops tools stay aligned with Bulqit records.

**Zoho Desk/Support** — A separate package (`zoho-support-client`) targets the Zoho Desk API for support ticket management. Inbound support emails are routed through the webhook endpoint.

**Features controlled:** CRM alignment for leads/contacts/deals, and support ticket routing.
