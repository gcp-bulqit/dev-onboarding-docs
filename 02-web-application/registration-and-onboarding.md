# Registration and onboarding

## Paths

| Flow | Path |
|------|------|
| Homeowner onboarding | `apps/web/src/app/onboarding/homeowner/` |
| Vendor onboarding | `apps/web/src/app/onboarding/vendor/` |
| Onboarding layout | `apps/web/src/app/onboarding/layout.tsx` |
| Homeowner step components | `apps/web/src/components/homeowners/onboarding/` |
| Vendor step components | `apps/web/src/components/vendors/onboarding/` |

## Homeowner onboarding steps

| Route | Step |
|-------|------|
| `/onboarding/homeowner/` | Intro |
| `/onboarding/homeowner/account-setup` | Account setup (feature-flagged via `isPhaseTwoOnboardingEnabled()`) |
| `/onboarding/homeowner/contact-info` | Contact information |
| `/onboarding/homeowner/address` | Address search |
| `/onboarding/homeowner/interest` | Service interest |
| `/onboarding/homeowner/service-selection` | Service selection |
| `/onboarding/homeowner/pricing` | Pricing |
| `/onboarding/homeowner/verification` | Verification |
| `/onboarding/homeowner/loading` | Processing / loading |
| `/onboarding/homeowner/confirmation` | Confirmation |
| `/onboarding/homeowner/finish` | Finish |

## Vendor onboarding steps

| Route | Step |
|-------|------|
| `/onboarding/vendor/` | Welcome |
| `/onboarding/vendor/account` | Account |
| `/onboarding/vendor/contact` | Contact |
| `/onboarding/vendor/basics` | Basics |
| `/onboarding/vendor/business` | Business name |
| `/onboarding/vendor/company` | Company details |
| `/onboarding/vendor/services` | Services |
| `/onboarding/vendor/verification` | Verification |
| `/onboarding/vendor/admin` | Admin review |
| `/onboarding/vendor/loading` | Processing / loading |

## Behavior synopsis

Route segments under **`onboarding/`** are **multi-step wizards**: each `page.tsx` mounts a **step component** and may load **Sanity-driven copy** for titles and descriptions.

The entire onboarding flow is gated by a phase-two feature flag in `onboarding/layout.tsx`, which checks `NEXT_PUBLIC_PHASE_TWO_FEATURE_FLAG` / `PHASE_TWO_FEATURE_FLAG` env vars and redirects to the landing page if not enabled.

Example — `onboarding/homeowner/account-setup/page.tsx`:

- **`generateMetadata`** — `sanityFetch` with **`homeownerOnboardingAccountQuery`** for SEO title/description from CMS.
- **Page** — Renders **`AccountSetupStep`** directly.

## Known issues

- `apps/web/src/components/homeowners/onboarding/steps/AddressStep.tsx` is a 23-line placeholder that is not used. The real implementation is `AddressSearchStep.tsx`.
- `apps/web/src/consts/vendorOnboardingSteps.ts` uses the `accountStep` ID for two different steps (lines 100 and 111), which is a routing collision.

**Features controlled:** Account creation funnels for **homeowners** vs **vendors** (profile, services, verification, pricing), gated by flags and Clerk until the user can access dashboards.
