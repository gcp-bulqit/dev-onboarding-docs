# Vendor dashboard

## Paths

| Item | Path |
|------|------|
| Layout (server) | `apps/web/src/app/dashboard/vendor/layout.tsx` |
| Shell | `apps/web/src/components/vendors/dashboard/VendorDashboardLayout.tsx` |
| Routes | `apps/web/src/app/dashboard/vendor/` — `bidding/`, `my-jobs/`, `calendar/`, `homes/`, `ratings/`, `reports/`, `inbox/`, `settings/`, etc. |
| Vendor APIs (examples) | `apps/web/src/app/api/vendor/me/`, `export-jobs/`, `transactions/` |

## Behavior synopsis

### `layout.tsx` (server)

- **`auth()`** + **`redirectIfNeeded`** — Same pattern as member layout for vendor URLs.
- **`getCurrentVendorTeamMember()`** — Loads **vendor** + **team member** row; distinguishes team member name vs vendor owner name.
- **Clerk `users.getUser(userId)`** — Profile image for the signed-in user.
- Resolves **`adminName`** when **`actor`** (impersonation) is present.
- Passes **`role: VendorTeamMemberRole`** and branding fields into **`VendorDashboardLayout`**; on vendor-not-found style errors, redirects to sign-in with **`auth_error`** (see file for exact `HttpError` handling).

### `VendorDashboardLayout.tsx` (client)

- Wraps with **`BottomNavProvider`** and **`TwilioProvider`** (vendors use Twilio for conversations).
- Uses **`useConversations`** with vendor identity, same pattern as member layout for **inbox** and message previews.
- **`canAccessFeature` / `VendorPermission` / `VendorTeamMemberRole`** — Hides or shows nav items (bidding, reports, settings tabs) based on **team role** from `getVendorSettingsTabs` and permission helpers.
- Sidebar links cover **home, bidding, jobs, calendar, homes, ratings, reports (transactions/disbursements), inbox, settings** (see `@/consts/urls` imports in file).

**Features controlled:** Vendor operations—**bidding** on blocks, **job** execution and exports, **financial reports**, **Stripe/payout** settings, **team** permissions, **ratings**, and **Twilio** messaging.
