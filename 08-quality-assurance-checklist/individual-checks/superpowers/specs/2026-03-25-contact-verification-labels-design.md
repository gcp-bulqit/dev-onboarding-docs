# Contact Verification Labels & Triggers

**Date:** 2026-03-25
**Status:** Approved
**Scope:** Vendor Settings (My Profile) + Member Settings (My Account)

## Problem

1. Vendor My Profile shows "Verified" badge on **phone** using `!!phone` (presence-based) — incorrect. A phone number existing in the DB doesn't mean it was verified by Clerk.
2. No "Verified" badge shown for **email** on vendor profile, despite `emailVerificationStatus` existing in the DB.
3. Member Settings shows no verification badges at all — only warning banners when email/phone are missing.
4. No way to trigger verification for contacts that exist but weren't verified (e.g., migrated data).

## Decisions

| Decision                                | Choice                                          | Rationale                                                  |
| --------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------- |
| Source of truth for verification status | **Clerk**                                       | Single source via `useUser()`, no DB sync issues           |
| Email verification mechanism            | **SendGrid OTP (server actions)**               | Maintain existing vendor flow, control over templates      |
| Phone verification mechanism            | **Clerk SDK (client-side)**                     | Existing `usePhoneVerification` hook, SMS via Clerk        |
| Badge/trigger placement                 | **Inline in field label**                       | e.g., "Phone number · Verified" or "Phone number · Verify" |
| OTP dialog                              | **Shared reusable component**                   | Extracted from existing VendorProfileForm code             |
| Member email verification               | **Create new server actions**                   | Mirror vendor flow with shared utility functions           |
| Verification trigger timing             | **On save (existing) + on-demand button (new)** | Button covers unverified legacy contacts                   |

## Architecture

### 1. Hook: `useVerificationStatus`

**File:** `apps/web/src/hooks/useVerificationStatus.ts`

```ts
function useVerificationStatus() {
  const { user } = useUser()

  const isEmailVerified = user?.primaryEmailAddress?.verification?.status === 'verified'
  const isPhoneVerified = user?.primaryPhoneNumber?.verification?.status === 'verified'

  return { isEmailVerified, isPhoneVerified }
}
```

- Replaces server-provided `isEmailVerified` / `isPhoneVerified` props
- After OTP verification, call `user.reload()` to update state
- Used in both vendor and member settings forms

### 2. Component: `OtpVerificationDialog`

**File:** `apps/web/src/components/shared/OtpVerificationDialog.tsx`

```ts
interface OtpVerificationDialogProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  type: 'email' | 'phone'
  contact: string
  onSendCode: () => Promise<void>
  onVerifyCode: (code: string) => Promise<boolean>
  onSuccess: () => void
}
```

**Behavior:**

- Dynamic title: "Verify your email" / "Verify your phone number"
- Description: "Enter the verification code sent to {contact}"
- 6-digit OTP input (same visual pattern as existing)
- "Resend code" button with 60s cooldown
- States: idle → sending → awaiting_code → verifying → success/error
- Caller provides `onSendCode` and `onVerifyCode` callbacks — dialog is agnostic to email vs phone implementation

**Extraction:** This is extracted from existing inline OTP dialog code in VendorProfileForm, not built from scratch.

### 3. Component: `VerificationBadge`

**File:** `apps/web/src/components/shared/VerificationBadge.tsx`

```ts
interface VerificationBadgeProps {
  verified: boolean
  onVerify?: () => void
}
```

**Rendering:**

- `verified=true`: Green `CheckCircleIcon` + "Verified" text in `text-primary-teal` (existing visual pattern)
- `verified=false` with `onVerify`: Blue "Verify" link in `text-primary-blue`, clickable
- `verified=false` without `onVerify`: renders nothing

### 4. Server Actions: Member Email Verification

**New files:**

- `apps/web/src/actions/dashboard/member/sendMemberEmailVerificationAction.ts`
- `apps/web/src/actions/dashboard/member/verifyMemberEmailCodeAction.ts`

**Shared utility (new):**

- `apps/web/src/lib/email-verification/emailVerificationUtils.ts`
  - `generateAndSendOtp(table, userId, email)` — generates 6-digit OTP, stores in DB, sends via SendGrid
  - `verifyOtp(table, userId, code)` — validates OTP, updates `emailVerificationStatus` to `VALID`, clears code, calls `syncEmailToClerk()`

**Flow:**

1. `sendMemberEmailVerificationAction` → calls `generateAndSendOtp(homeowners, ...)`
2. `verifyMemberEmailCodeAction` → calls `verifyOtp(homeowners, ...)`

Existing vendor actions will be refactored to use the same shared utilities.

### 5. Changes to Existing Forms

#### VendorProfileForm (`components/vendors/dashboard/settings/VendorProfileForm.tsx`)

- Remove `isEmailVerified` / `isPhoneVerified` props from server page
- Use `useVerificationStatus()` hook instead
- Replace inline OTP dialog with `OtpVerificationDialog`
- Add `VerificationBadge` to email field label (currently missing)
- Fix phone field label to use Clerk status instead of `!!phone`
- Simplify server page that previously computed verification props

#### MemberAccountForm (`components/members/settings/MemberAccountForm.tsx`)

- Add `useVerificationStatus()` hook
- Add `VerificationBadge` to email and phone field labels
- Integrate `OtpVerificationDialog` for both email and phone
- Phone: use existing `usePhoneVerification` hook as callback
- Email: use new `sendMemberEmailVerificationAction` / `verifyMemberEmailCodeAction` as callbacks
- Remove yellow warning banners (replaced by inline badge)

### 6. Verification Flows

#### On Save (existing behavior, maintained)

1. User changes email/phone → saves form
2. OTP dialog opens automatically
3. User enters code → verification succeeds
4. `user.reload()` updates badge to "Verified"

#### On-Demand Button (new)

1. Contact exists but is not verified (e.g., migrated data)
2. User clicks "Verify" link in field label
3. OTP dialog opens
4. User enters code → verification succeeds
5. `user.reload()` updates badge to "Verified"

## Edge Cases

1. **Form field is empty** — no badge or button shown (nothing to verify)
2. **Form field has value, Clerk says unverified (or contact not in Clerk)** — "Verify" button appears. Clicking it creates the contact in Clerk if needed, then sends OTP
3. **Form field has value, Clerk says verified** — "Verified" badge shown
4. **OTP expired** — "Resend code" button in dialog, 60s cooldown
5. **User closes dialog without verifying** — contact stays unverified, badge still shows "Verify"
6. **Send failure (SMS/email)** — dialog shows error message, allows retry

## Testing

- **`useVerificationStatus`** — unit tests mocking `useUser` with various Clerk states (verified, unverified, missing)
- **`VerificationBadge`** — unit tests for conditional rendering (badge vs button vs nothing)
- **`OtpVerificationDialog`** — unit tests for full state machine (idle → sending → awaiting → verifying → success/error)
- **Existing form tests** — adapt `VendorProfileForm` and `MemberAccountForm` tests to new hook-based verification logic
- **Server actions** — unit tests for `generateAndSendOtp` and `verifyOtp` shared utilities

## No Changes Required

- Database schema (fields already exist: `emailVerificationStatus`, `emailVerificationCode` in both tables)
- `usePhoneVerification` hook (reused as-is)
- SendGrid email templates
- `syncEmailToClerk` utility

## Files Summary

| File                                                                | Action                                        |
| ------------------------------------------------------------------- | --------------------------------------------- |
| `hooks/useVerificationStatus.ts`                                    | **Create**                                    |
| `components/shared/OtpVerificationDialog.tsx`                       | **Create** (extracted from VendorProfileForm) |
| `components/shared/VerificationBadge.tsx`                           | **Create**                                    |
| `lib/email-verification/emailVerificationUtils.ts`                  | **Create** (shared OTP logic)                 |
| `actions/dashboard/member/sendMemberEmailVerificationAction.ts`     | **Create**                                    |
| `actions/dashboard/member/verifyMemberEmailCodeAction.ts`           | **Create**                                    |
| `components/vendors/dashboard/settings/VendorProfileForm.tsx`       | **Modify**                                    |
| `components/members/settings/MemberAccountForm.tsx`                 | **Modify**                                    |
| `app/dashboard/vendor/settings/my-profile/page.tsx` (or equivalent) | **Modify** (remove verification props)        |
| Existing vendor email verification actions                          | **Modify** (refactor to use shared utils)     |
