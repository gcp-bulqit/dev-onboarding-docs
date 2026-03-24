# `packages/subscription-billing`

## Paths

| Item | Path |
|------|------|
| Pricing | `packages/subscription-billing/src/pricing.ts` — `calculateSamPricing`, `resolveSamMetricValue` |
| Coupons | `packages/subscription-billing/src/coupons.ts` — `saveCouponToHomeowner`, `listSavedCoupons`, `resolveBestSavedCouponForCharge`, `applySavedCouponUsage`, `removeSavedCouponsByIds` |
| Credits | `packages/subscription-billing/src/credits.ts` — `getAvailableHomeownerCreditBalanceCents`, `consumeHomeownerCredits` |

## Behavior synopsis

Pure **domain logic** for **SAM-style pricing**, **promo codes**, and **homeowner credits** with explicit **`DbClient` / `DbTransaction`** typing—no UI.

**Features controlled:** Checkout line-item pricing, coupon stacking rules, and applying **account credits** before charging cards (used from server actions and APIs touching **`/dashboard/member/checkout`**).
