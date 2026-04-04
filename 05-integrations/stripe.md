# Stripe

## SDK packages

| Package | Usage |
|---------|-------|
| `stripe` | Server-side SDK, used in `apps/web/src/lib/stripe/` |
| `@stripe/stripe-js` | Client-side Stripe.js for payment confirmation |
| `@stripe/react-stripe-js` | React components for Stripe Elements |

## Key areas

- **Server SDK wrapper** — `apps/web/src/lib/stripe/` handles payment intents, payment methods, Connect accounts, and webhook verification.
- **Webhook handler** — `apps/web/src/app/api/webhooks/stripe/route.ts` processes payment, subscription, and payout events.
- **Payment and dunning library** — `apps/web/src/lib/payments/` and `apps/web/src/lib/payouts/`
- **Billing domain package** — `packages/subscription-billing/` for pricing, coupons, and credits.
- **Temporal workflows** — Payment dunning, subscription billing, and weekly payouts run as scheduled workflows.

For the full server action inventory see [payments.md](../02-web-application/payments.md).

**Features controlled:** Homeowner checkout, payment method management, vendor Connect onboarding and payouts, dunning and failure recovery, subscription billing, and financial reporting.
