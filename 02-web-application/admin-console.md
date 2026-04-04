# Admin console

## Paths

| Item | Path |
|------|------|
| Root layout | `apps/web/src/app/admin/layout.tsx` |
| Navigation | `apps/web/src/components/admin/ui/layout/AdminSidebar.tsx` |
| Route tree | `apps/web/src/app/admin/` |

## Route groups

| Route | Domain |
|-------|--------|
| `members/` | Member management |
| `vendors/` | Vendor management |
| `lots/` | Lot management |
| `blocks/` | Block management |
| `services/` | Service configuration |
| `service-components/` | Service component configuration |
| `service-opportunities/` | Service opportunity tracking |
| `orders/` | Order management |
| `jobs/` | Job management |
| `bidding/` | Bidding management |
| `financials/` | Financial overview |
| `transactions/` | Transaction history |
| `payouts/` | Payout management |
| `coupons/` | Coupon management |
| `notifications/` | Notification templates and configuration |
| `messages/` | Messaging admin |
| `reviews/` | Review management |
| `referral-campaigns/` | Referral campaign management |
| `vendor-promotions/` | Vendor promotion management |
| `categories/` | Service categories |
| `metro-areas/` | Metro area configuration |
| `coffee-shops/` | Coffee shop locations |
| `settings/` | Admin settings |
| `alerts/` | Dashboard alerts |
| `metrics/` | Metrics |
| `audit-trail/` | Audit trail |
| `archive/` | Archived items |
| `trash/` | Deleted items |
| `dev/` | Developer tools (messaging simulation, diagnostics) |
| `temporal-test/` | Temporal workflow test page |
| `sign-in/` | Admin sign-in |

## Behavior synopsis

### `layout.tsx`

- **`isUserAdmin()`** — Resolves Clerk **role** from session or full user metadata; returns whether user is **`ADMIN_ROLE`**.
- **Fonts** — Loads Geist / Geist Mono for admin chrome.
- **If admin** — Calls **`ensureAdminSynced(userId)`** (`@/lib/admin/currentAdmin`) to mirror the Clerk user into the app DB for admin features.
- Wraps content in **`ClerkProvider`**. When **`isAdmin`**, **`SignedIn`** renders **`AutoSelectOrganization`** (Clerk org context for admin) and **`AdminSidebar`**; **`main`** always renders **`children`** so sign-in pages can show without the shell.
- **Toasters** — Radix **`Toaster`** + **Sonner** for admin feedback.

### Child routes

Each subdirectory's `page.tsx` / server actions implement CRUD for that domain. Admin **API routes** under `apps/web/src/app/api/admin/` back exports, batch jobs, and diagnostics.

### Large files

`apps/web/src/app/admin/vendors/[id]/VendorDetailPage.tsx` is 3,503 lines. It contains stats display, tabs, forms, modals, and tables for vendor management. It has one remaining TODO (activity log, line 3208).

### Known gaps

- The `vendor_jobs` database table does not exist. No migration creates it.

**Features controlled:** Internal operations — marketplace configuration, user and vendor management, money movement (payouts/transactions), communications (notifications + messaging tooling), and imports/exports.
