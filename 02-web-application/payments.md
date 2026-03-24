# Payments

## Paths

| Item | Path |
|------|------|
| Stripe return / confirm | `apps/web/src/app/payment/confirm/page.tsx` |
| Member checkout | `apps/web/src/app/dashboard/member/checkout/` |
| Vendor payouts / banking | `apps/web/src/app/dashboard/vendor/settings/payouts/` (and related) |
| Billing package (domain rules) | `packages/subscription-billing/` |

## Behavior synopsis — `payment/confirm/page.tsx`

- Client component using **`loadStripe`** (`@stripe/stripe-js`) with **`NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`**.
- **`useSearchParams`** reads **`setup_intent_client_secret`** / **`setup_intent`** or **`redirect_status`** (3DS / wallet returns).
- **`stripe.retrieveSetupIntent`** — Confirms success/failure; on success stores **`payment_method` id** in **`sessionStorage`** as `savedPaymentMethodId` and redirects to **`/dashboard/member/checkout/payment?payment_confirmed=true`** after a short delay.
- Renders loading/success/error UI with **`Card`** from the design system.

**Features controlled:** Saving a **default payment method** for homeowner checkout (SetupIntents) and returning users cleanly into the **checkout** flow.
