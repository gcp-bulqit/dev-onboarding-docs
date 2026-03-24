# Clerk

## Paths

| Item | Path |
|------|------|
| Middleware | `apps/web/src/proxy.ts` — `clerkMiddleware`, `auth()`, `clerkClient()` |
| Root providers | `apps/web/src/components/providers/AuthProviders.tsx` — `ClerkProvider` |
| Admin layout | `apps/web/src/app/admin/layout.tsx` — `ClerkProvider`, `auth()` |
| Docs | `docs/clerk-manual-user-setup.md` |

## Behavior synopsis

- **Sessions & JWT claims** — Custom **`metadata.role`** drives routing (admin vs member vs vendor); middleware and layouts re-fetch **`users.getUser`** when claims are missing.
- **`ClerkProvider`** — Supplies client hooks (`useAuth`, `useUser`, `useClerk`) to dashboard components; **`afterSignOutUrl`** sends users to admin sign-in from `AuthProviders`.
- **Impersonation** — Clerk **`actor`** on the session lets admins use member/vendor UIs safely (see middleware + layouts).

**Features controlled:** All **login-gated** experiences—dashboards, admin, and secure API contexts that call **`auth()`**.
