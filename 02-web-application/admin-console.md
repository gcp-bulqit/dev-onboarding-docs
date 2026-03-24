# Admin console

## Paths

| Item | Path |
|------|------|
| Root layout | `apps/web/src/app/admin/layout.tsx` |
| Navigation | `apps/web/src/components/admin/ui/layout/AdminSidebar.tsx` |
| Route tree | `apps/web/src/app/admin/` — members, vendors, orders, financials, notifications, messaging, jobs, bidding, etc. |

## Behavior synopsis

### `layout.tsx`

- **`isUserAdmin()`** — Resolves Clerk **role** from session or full user metadata; returns whether user is **`ADMIN_ROLE`**.
- **Fonts** — Loads Geist / Geist Mono for admin chrome.
- **If admin** — Calls **`ensureAdminSynced(userId)`** (`@/lib/admin/currentAdmin`) to mirror the Clerk user into the app DB for admin features.
- Wraps content in **`ClerkProvider`**. When **`isAdmin`**, **`SignedIn`** renders **`AutoSelectOrganization`** (Clerk org context for admin) and **`AdminSidebar`**; **`main`** always renders **`children`** so sign-in pages can show without the shell.
- **Toasters** — Radix **`Toaster`** + **Sonner** for admin feedback.

### Child routes

Each subdirectory’s `page.tsx` / server actions implement CRUD for that domain (members, payouts, service opportunities, etc.). Admin **API routes** under `apps/web/src/app/api/admin/` back exports, batch jobs, and diagnostics.

**Features controlled:** Internal operations—marketplace configuration, user and vendor management, money movement (payouts/transactions), communications (notifications + messaging tooling), and imports/exports.
