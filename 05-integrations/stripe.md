# Stripe

## Paths

| Item | Path |
|------|------|
| Payment return | `apps/web/src/app/payment/confirm/page.tsx` — `loadStripe`, `retrieveSetupIntent` |
| APIs | `apps/web/src/app/api/vendor/transactions/` — invoices / Stripe helpers |
| Domain | `packages/subscription-billing/` — pricing, coupons, credits |

## Behavior synopsis

**Stripe.js** on the client confirms **SetupIntents** (saved payment methods) and handles wallet/3DS **`redirect_status`** returns. Server routes use Stripe’s server SDK (where configured) for **Connect** payouts, invoices, and transaction exports.

**Features controlled:** **Homeowner checkout** (cards + wallets) and **vendor payouts** / financial reporting.
