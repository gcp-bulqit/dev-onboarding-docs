# Registration and onboarding

## Paths

| Flow | Path |
|------|------|
| Homeowner registration | `apps/web/src/app/register/homeowner/` |
| Other register segments | `apps/web/src/app/register/` |
| Homeowner onboarding | `apps/web/src/app/onboarding/homeowner/` |
| Vendor onboarding | `apps/web/src/app/onboarding/vendor/` |
| Homeowner UI | `apps/web/src/components/homeowners/onboarding/`, `homeowners/registration/` |
| Vendor UI | `apps/web/src/components/vendors/onboarding/` |

## Behavior synopsis

Route segments under **`register/`** and **`onboarding/`** are **multi-step wizards**: each `page.tsx` mounts a **step component** and may load **Sanity-driven copy** for titles/descriptions.

Example — `onboarding/homeowner/account-setup/page.tsx`:

- **`generateMetadata`** — `sanityFetch` with **`homeownerOnboardingAccountQuery`** for SEO title/description from CMS.
- **Page** — Reads **`isPhaseTwoOnboardingEnabled()`** (feature flag) and passes it to **`AccountSetupStep`** so the UI can branch.

**Features controlled:** Account creation funnels for **homeowners** vs **vendors** (profile, services, verification, pricing), gated by flags and Clerk until the user can access dashboards.
