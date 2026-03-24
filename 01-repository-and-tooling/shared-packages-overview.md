# Shared packages overview

Workspace packages live under `packages/`. Each has its own `package.json` and is linked with `workspace:*` from apps.

## Index

| Package directory | Primary responsibility |
|-------------------|-------------------------|
| `packages/database/` | Drizzle schemas, migrations, DB instance factories (`instances/node`, `instances/serverless`, etc.). |
| `packages/sanity-cms/` | Sanity config, server/client, GROQ, CLI, generated types. |
| `packages/notifications/` | Event names, queue delivery (`claimDueQueueItems`, `deliverQueueItem`), template resolution. |
| `packages/messaging/` | `MessagingServiceCore`, Twilio Conversations gateway, phone pool, vendor contact helpers. |
| `packages/temporal-workflows/` | Workflow + activity implementations (billing, notifications, payouts, vendor scores, etc.). |
| `packages/zoho-client/` | OAuth refresh + CRM API (`ZohoClient`). |
| `packages/zoho-support-client/` | Support-ticket API client. |
| `packages/referral-factory-client/` | Referral Factory HTTP client and types. |
| `packages/subscription-billing/` | SAM pricing, coupons, homeowner credits. |
| `packages/vendors-bulqit-score/` | Bayesian / composite vendor score math. |
| `packages/display-ids/` | `formatDisplayId`, table prefix map. |
| `packages/utilities/` | Small shared modules: `strings`, `numbers`, `environment`, `booleans`. |
| `packages/eslint-config/` | Flat ESLint presets (`node`, `next`). |

**Behavior synopsis:** These packages are **imported** by `apps/web` and workers; they contain no routes. They centralize domain rules so the Next app and workers stay consistent (same DB types, same notification events, same Temporal definitions).

Deeper per-package notes: [../04-shared-packages/](../04-shared-packages/).
