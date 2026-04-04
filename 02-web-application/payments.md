# Payments

## Paths

| Item | Path |
|------|------|
| Stripe return / confirm | `apps/web/src/app/payment/confirm/page.tsx` |
| Member checkout | `apps/web/src/app/dashboard/member/checkout/` |
| Vendor payouts / banking | `apps/web/src/app/dashboard/vendor/settings/payouts/` (and related) |
| Stripe webhook | `apps/web/src/app/api/webhooks/stripe/route.ts` |
| Vendor invoice (Stripe) | `apps/web/src/app/api/vendor/transactions/[id]/invoice/stripe/route.ts` |
| Stripe server SDK wrapper | `apps/web/src/lib/stripe/` |
| Payment / dunning library | `apps/web/src/lib/payments/` |
| Payout batch library | `apps/web/src/lib/payouts/` |
| Billing package (domain rules) | `packages/subscription-billing/` |

## Server actions

| Area | Path | Key actions |
|------|------|-------------|
| Admin payments | `actions/admin/payments/` | Transactions, refunds, credits, payouts, financial metrics, export |
| Member checkout | `actions/member/checkout/` | createPaymentIntent, createSetupIntent, processCheckout, applyCoupon, validateCoupon, createOrder |
| Member payment methods | `actions/member/payment-methods/` | add, remove, setDefault, getPaymentMethods |
| Vendor Connect | `actions/vendors/connect/` | startConnectOnboarding, checkConnectStatus, getStripeDashboardLink, refreshOnboardingLink |
| Vendor payouts | `actions/vendors/payouts/` | getVendorPayouts, getPayoutDetails, getVendorBalance, setupPayoutMethod |

## Stripe webhook (`api/webhooks/stripe/route.ts`)

Handles incoming Stripe events for payment processing, subscription lifecycle, and payout tracking. Validates webhook signatures using the Stripe SDK directly (`getStripeClient().webhooks.constructEvent`).

## Temporal workflows

Three payment-related workflows run on the Temporal worker:

| Workflow | Path | Purpose |
|----------|------|---------|
| payment-dunning | `packages/temporal-workflows/src/payment-dunning/` | Retry failed payments with escalating dunning |
| subscription-billing | `packages/temporal-workflows/src/subscription-billing/` | Recurring billing cycle processing |
| weekly-payouts | `packages/temporal-workflows/src/weekly-payouts/` | Batch vendor payout processing |

## Stripe server SDK (`lib/stripe/`)

| File | Purpose |
|------|---------|
| `client.ts` | Stripe SDK initialization |
| `config.ts` | Stripe configuration |
| `connect.ts` | Stripe Connect (vendor onboarding, account management) |
| `payment-intents.ts` | PaymentIntent creation and management |
| `payment-methods.ts` | Payment method CRUD |
| `webhooks.ts` | Webhook signature verification |
| `dashboard-url.ts` | Stripe dashboard link generation |
| `types.ts` | Shared Stripe type definitions |

## Payment library (`lib/payments/`)

| File | Purpose |
|------|---------|
| `dunning-scheduler.ts` | Schedule retry attempts for failed payments |
| `dunning.ts` | Dunning logic and escalation |
| `failure-classification.ts` | Classify payment failure types for retry decisions |
| `triggerServiceJobPayment.ts` | Initiate payment for completed service jobs |
| `homeownerCoupons.ts` | Coupon application logic |
| `homeownerCredits.ts` | Credit balance management |
| `transactionDisplay.ts` | Transaction formatting for UI |

## Behavior synopsis — `payment/confirm/page.tsx`

- Client component using **`loadStripe`** (`@stripe/stripe-js`) with **`NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`**.
- **`useSearchParams`** reads **`setup_intent_client_secret`** / **`setup_intent`** or **`redirect_status`** (3DS / wallet returns).
- **`stripe.retrieveSetupIntent`** — Confirms success/failure; on success stores **`payment_method` id** in **`sessionStorage`** as `savedPaymentMethodId` and redirects to **`/dashboard/member/checkout/payment?payment_confirmed=true`** after a short delay.
- Renders loading/success/error UI with **`Card`** from the design system.

**Features controlled:** Saving a **default payment method** for homeowner checkout (SetupIntents) and returning users cleanly into the **checkout** flow.
