# Payments

## Paths

| Item | Path |
|------|------|
| Stripe return / confirm | `apps/web/src/app/payment/confirm/page.tsx` |
| Member checkout | `apps/web/src/app/dashboard/member/checkout/` |
| Vendor payouts / banking | `apps/web/src/app/dashboard/vendor/settings/payouts/` (and related) |
| Stripe webhook | `apps/web/src/app/api/webhooks/stripe/route.ts` |
| Stripe server SDK wrapper | `apps/web/src/lib/stripe/` |
| Payment / dunning library | `apps/web/src/lib/payments/` |
| Payout batch library | `apps/web/src/lib/payouts/` |
| Billing package (domain rules) | `packages/subscription-billing/` |

## Server actions

| Area | Path | Scope |
|------|------|-------|
| Admin payments | `actions/admin/payments/` | Transactions, refunds, credits, payouts, financial metrics, export |
| Member checkout | `actions/member/checkout/` | Payment intents, coupons, credits, order processing |
| Member payment methods | `actions/member/payment-methods/` | Add, remove, set default |
| Vendor Connect | `actions/vendors/connect/` | Stripe Connect onboarding, dashboard links, payout methods |
| Vendor payouts | `actions/vendors/payouts/` | Payout history, balance, disbursement setup |

## Stripe webhook

Handles incoming Stripe events for payment processing, subscription lifecycle, and payout tracking. Validates webhook signatures using the Stripe SDK directly.

## Temporal workflows

Three payment-related workflows run on the Temporal worker: payment dunning (retry failed payments with escalation), subscription billing (recurring cycle processing), and weekly payouts (batch vendor disbursements).

## Key libraries

- **`lib/stripe/`** — Server-side Stripe SDK wrapper for payment intents, payment methods, Connect accounts, and webhook verification.
- **`lib/payments/`** — Dunning scheduler, failure classification, payment triggering, coupons, and credits.
- **`lib/payouts/`** — Payout batch processing and scheduling.

## Behavior synopsis — `payment/confirm/page.tsx`

- Client component using **`loadStripe`** with **`NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`**.
- Reads `setup_intent_client_secret` / `redirect_status` from search params to confirm SetupIntents (saved payment methods, 3DS / wallet returns).
- On success, stores the payment method ID in `sessionStorage` and redirects to the checkout flow.

**Features controlled:** Homeowner checkout, payment method management, vendor Connect onboarding and payouts, dunning and failure recovery, subscription billing, and financial reporting.
