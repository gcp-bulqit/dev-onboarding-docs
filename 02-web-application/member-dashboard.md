# Member (homeowner) dashboard

## Paths

| Item | Path |
|------|------|
| Layout (server) | `apps/web/src/app/dashboard/member/layout.tsx` |
| Shell (client) | `apps/web/src/components/members/dashboard/MemberDashboardLayout.tsx` |
| Feature routes | `apps/web/src/app/dashboard/member/` — `services/`, `checkout/`, `settings/`, `inbox/`, etc. |
| Member APIs (examples) | `apps/web/src/app/api/dashboard/member/jobs/rate/`, `apps/web/src/app/api/member/invoices/` |

## Behavior synopsis

### `layout.tsx` (server)

- **`auth()`** — Reads `userId`, `sessionClaims.metadata.role`, **`actor`** (impersonation).
- **`redirectIfNeeded`** — Sends unauthenticated or wrong-role users to dashboard sign-in.
- **`getCurrentMember()`** — Loads the DB member; on **`HttpError`** with “member not found”, redirects with **`auth_error`** query for explicit UX.
- **Clerk `users.getUser`** — Fetches avatar URL when `clerkUserId` exists; if impersonating, resolves **admin display name** from `actor.sub`.
- Renders **`MemberDashboardLayout`** with **`memberName`**, **`memberImageUrl`**, **`isImpersonating`**, **`adminName`**.

### `MemberDashboardLayout.tsx` (client)

- Wraps children in **`MemberProviders`**, **`BottomNavProvider`**, **`InboxCountsProvider`**.
- Integrates **Twilio Conversations** via **`useTwilioContext`**, **`useConversations('homeowner')`**, **`useConversationMessageSubscriptions`** for inbox previews and unread handling (with debounced refresh).
- Renders **sidebar** (`SidebarDesktop` / `SidebarMobile`), **impersonation banner**, **toast** host, and navigation to services, inbox, settings, referrals, etc. (URLs from `@/consts/urls`).

**Features controlled:** Homeowner home, services catalog/checkout, notifications and **messaging inbox**, account settings, job rating (via API), and **admin impersonation** UX.
