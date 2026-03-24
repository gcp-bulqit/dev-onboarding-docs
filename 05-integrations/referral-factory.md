# Referral Factory

## Paths

| Item | Path |
|------|------|
| Client lib | `packages/referral-factory-client/` |
| Admin UI | `apps/web/src/app/admin/referral-campaigns/` |
| Public routes | `/refer` — listed as public in `apps/web/src/proxy.ts` |

## Behavior synopsis

Server-side calls use the **API key** to register participants and read campaign state for **referral links** and **admin reporting**.

**Features controlled:** External **referral program** mechanics layered on Bulqit sign-up flows.
