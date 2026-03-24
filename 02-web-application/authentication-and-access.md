# Authentication and access

## Paths

| Concern | Path |
|---------|------|
| Clerk middleware | `apps/web/src/proxy.ts` |
| Client providers | `apps/web/src/components/providers/AuthProviders.tsx` |
| Role / URL constants | `apps/web/src/consts/roles.ts`, `apps/web/src/consts/urls.ts` |
| Sign-in (marketing) | `apps/web/src/app/(public)/sign-in/[[...sign-in]]/` |
| Dashboard sign-in | `apps/web/src/app/dashboard/sign-in/` |
| Admin sign-in | `apps/web/src/app/admin/sign-in/` |
| OAuth callback | `apps/web/src/app/oauth-callback/`, `apps/web/src/components/oauth-callback/` |
| Activate account | `apps/web/src/app/activate-account/` |
| Account type selection | `apps/web/src/app/account-type-selection/` |
| Impersonation | `apps/web/src/app/impersonate/`, `apps/web/src/components/shared/ImpersonationBanner.tsx` |
| Error pages | `apps/web/src/app/401/`, `403/`, `server-error/` |

## Behavior synopsis — `proxy.ts` (Clerk middleware)

- **`createRouteMatcher`** builds predicates: **protected** (`/admin`, `/dashboard/vendor|homeowner|member`), **API** (`/api/admin` — no `auth.protect()` here), **public exceptions** (sign-in pages), **fully public** marketing URLs (skips Clerk entirely for latency).
- **`clerkMiddleware` callback** — On public routes: returns `NextResponse.next()` + **`X-Robots-Tag`**. Else calls **`auth()`** for `userId`, **`sessionClaims`**, **`actor`** (impersonation).
- **Role resolution** — Reads `sessionClaims.metadata.role`; if absent, **`clerkClient().users.getUser`** merges public/unsafe/private metadata into a single role string.
- **Protected HTML routes** — **`auth.protect()`** then **role checks**: admin-only under `ADMIN_HOME_URL`; member routes allow member/homeowner or **impersonating** admin; other dashboards allow vendor (+ team member), homeowner, member, or impersonation.
- **`redirect_url` cookie** — After login on dashboard/admin, reads httpOnly cookie and redirects once to deep links (with validation to `/dashboard` or `/admin` only).
- **Sign-in pages** — If `userId` present and no `auth_error`, handles **`redirect_url` query** (cookie) or **role-based redirect** (admin → admin home, homeowner/member → member dashboard, vendor roles → vendor dashboard).
- **`config.matcher`** — Excludes Twilio webhook, `api/cron`, static assets from middleware.

## `AuthProviders.tsx`

- Wraps the tree in **`ClerkProvider`** with **`afterSignOutUrl`** to admin sign-in, **`GlobalErrorHandler`**, and **`UserbackProvider`** (feedback widget).

## Related doc

- `docs/clerk-manual-user-setup.md`
