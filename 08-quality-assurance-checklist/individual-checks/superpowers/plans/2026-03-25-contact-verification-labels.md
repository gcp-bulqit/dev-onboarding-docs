# Contact Verification Labels & Triggers — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix verification badges to use Clerk as source of truth, add "Verify" trigger buttons for unverified contacts, and share OTP dialog between vendor and member settings.

**Architecture:** Extract shared `VerificationBadge` and `OtpVerificationDialog` components. Create `useVerificationStatus` hook that reads Clerk verification state. Refactor both `VendorProfileForm` and `MemberAccountForm` to use these shared components instead of inline OTP dialogs and incorrect verification checks.

**Tech Stack:** Next.js 15, Clerk SDK (`@clerk/nextjs`), React, TypeScript, Vitest, SendGrid (email OTP)

**Spec:** `docs/superpowers/specs/2026-03-25-contact-verification-labels-design.md`

---

## File Structure

| File                                                                                                     | Action | Responsibility                                          |
| -------------------------------------------------------------------------------------------------------- | ------ | ------------------------------------------------------- |
| `apps/web/src/hooks/useVerificationStatus.ts`                                                            | Create | Hook to read email/phone verification status from Clerk |
| `apps/web/src/hooks/_tests/useVerificationStatus.test.ts`                                                | Create | Tests for verification status hook                      |
| `apps/web/src/components/shared/VerificationBadge.tsx`                                                   | Create | Badge ("Verified") or trigger button ("Verify")         |
| `apps/web/src/components/shared/_tests/VerificationBadge.test.tsx`                                       | Create | Tests for badge component                               |
| `apps/web/src/components/shared/OtpVerificationDialog.tsx`                                               | Create | Reusable OTP dialog extracted from VendorProfileForm    |
| `apps/web/src/components/shared/_tests/OtpVerificationDialog.test.tsx`                                   | Create | Tests for OTP dialog                                    |
| `apps/web/src/lib/email-verification/emailVerificationUtils.ts`                                          | Create | Shared OTP generation + SendGrid sending + verification |
| `apps/web/src/lib/email-verification/_tests/emailVerificationUtils.test.ts`                              | Create | Tests for shared email verification utils               |
| `apps/web/src/actions/vendors/sendVendorEmailVerificationAction.ts`                                      | Modify | Refactor to use shared utils                            |
| `apps/web/src/actions/vendors/verifyVendorEmailCodeAction.ts`                                            | Modify | Refactor to use shared utils                            |
| `apps/web/src/actions/members/settings/sendEmailVerificationAction.ts`                                   | Modify | Refactor to use shared utils                            |
| `apps/web/src/actions/members/settings/verifyEmailCodeAction.ts`                                         | Modify | Refactor to use shared utils                            |
| `apps/web/src/components/vendors/dashboard/settings/VendorProfileForm.tsx`                               | Modify | Use shared components, fix verification logic           |
| `apps/web/src/app/dashboard/vendor/settings/profile/page.tsx`                                            | Modify | Remove `isEmailVerified`/`isPhoneVerified` props        |
| `apps/web/src/components/members/settings/MemberAccountForm.tsx`                                         | Modify | Use shared components, add badges                       |
| `apps/web/src/components/vendors/dashboard/settings/_tests/VendorProfileForm.phoneVerification.test.tsx` | Modify | Update for new hook/component                           |
| `apps/web/src/components/members/settings/_tests/MemberAccountForm.phoneVerification.test.tsx`           | Modify | Update for new hook/component                           |

---

## Task 1: Create `useVerificationStatus` hook

**Files:**

- Create: `apps/web/src/hooks/useVerificationStatus.ts`
- Create: `apps/web/src/hooks/_tests/useVerificationStatus.test.ts`

- [ ] **Step 1: Write the failing tests**

Create `apps/web/src/hooks/_tests/useVerificationStatus.test.ts`:

```ts
import { renderHook } from '@testing-library/react'
import { beforeEach, describe, expect, it, vi } from 'vitest'

const mockUser = {
  primaryEmailAddress: null as { verification?: { status?: string } } | null,
  primaryPhoneNumber: null as { verification?: { status?: string } } | null,
}

vi.mock('@clerk/nextjs', () => ({
  useUser: () => ({ user: mockUser }),
}))

import { useVerificationStatus } from '../useVerificationStatus'

describe('useVerificationStatus', () => {
  beforeEach(() => {
    mockUser.primaryEmailAddress = null
    mockUser.primaryPhoneNumber = null
  })

  it('returns false for both when user has no email or phone', () => {
    const { result } = renderHook(() => useVerificationStatus())

    expect(result.current.isEmailVerified).toBe(false)
    expect(result.current.isPhoneVerified).toBe(false)
  })

  it('returns true for email when primaryEmailAddress is verified', () => {
    mockUser.primaryEmailAddress = { verification: { status: 'verified' } }

    const { result } = renderHook(() => useVerificationStatus())

    expect(result.current.isEmailVerified).toBe(true)
    expect(result.current.isPhoneVerified).toBe(false)
  })

  it('returns true for phone when primaryPhoneNumber is verified', () => {
    mockUser.primaryPhoneNumber = { verification: { status: 'verified' } }

    const { result } = renderHook(() => useVerificationStatus())

    expect(result.current.isEmailVerified).toBe(false)
    expect(result.current.isPhoneVerified).toBe(true)
  })

  it('returns false for email when verification status is not verified', () => {
    mockUser.primaryEmailAddress = { verification: { status: 'unverified' } }

    const { result } = renderHook(() => useVerificationStatus())

    expect(result.current.isEmailVerified).toBe(false)
  })

  it('returns true for both when both are verified', () => {
    mockUser.primaryEmailAddress = { verification: { status: 'verified' } }
    mockUser.primaryPhoneNumber = { verification: { status: 'verified' } }

    const { result } = renderHook(() => useVerificationStatus())

    expect(result.current.isEmailVerified).toBe(true)
    expect(result.current.isPhoneVerified).toBe(true)
  })
})
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `cd apps/web && pnpm vitest run src/hooks/_tests/useVerificationStatus.test.ts`
Expected: FAIL — module `../useVerificationStatus` not found

- [ ] **Step 3: Implement the hook**

Create `apps/web/src/hooks/useVerificationStatus.ts`:

```ts
'use client'

import { useUser } from '@clerk/nextjs'

export function useVerificationStatus() {
  const { user } = useUser()

  const isEmailVerified = user?.primaryEmailAddress?.verification?.status === 'verified'
  const isPhoneVerified = user?.primaryPhoneNumber?.verification?.status === 'verified'

  return { isEmailVerified, isPhoneVerified }
}
```

- [ ] **Step 4: Run tests to verify they pass**

Run: `cd apps/web && pnpm vitest run src/hooks/_tests/useVerificationStatus.test.ts`
Expected: All 5 tests PASS

- [ ] **Step 5: Commit**

```bash
git add apps/web/src/hooks/useVerificationStatus.ts apps/web/src/hooks/_tests/useVerificationStatus.test.ts
git commit -m "feat: add useVerificationStatus hook reading Clerk state"
```

---

## Task 2: Create `VerificationBadge` component

**Files:**

- Create: `apps/web/src/components/shared/VerificationBadge.tsx`
- Create: `apps/web/src/components/shared/_tests/VerificationBadge.test.tsx`

- [ ] **Step 1: Write the failing tests**

Create `apps/web/src/components/shared/_tests/VerificationBadge.test.tsx`:

```tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { describe, expect, it, vi } from 'vitest'

import { VerificationBadge } from '../VerificationBadge'

describe('VerificationBadge', () => {
  it('renders "Verified" badge with check icon when verified=true', () => {
    render(<VerificationBadge verified={true} />)

    expect(screen.getByText('Verified')).toBeInTheDocument()
  })

  it('renders "Verify" button when verified=false and onVerify provided', () => {
    const onVerify = vi.fn()

    render(<VerificationBadge verified={false} onVerify={onVerify} />)

    expect(screen.getByRole('button', { name: 'Verify' })).toBeInTheDocument()
  })

  it('calls onVerify when "Verify" button is clicked', async () => {
    const user = userEvent.setup()
    const onVerify = vi.fn()

    render(<VerificationBadge verified={false} onVerify={onVerify} />)

    await user.click(screen.getByRole('button', { name: 'Verify' }))

    expect(onVerify).toHaveBeenCalledTimes(1)
  })

  it('renders nothing when verified=false and no onVerify', () => {
    const { container } = render(<VerificationBadge verified={false} />)

    expect(container.innerHTML).toBe('')
  })

  it('does not render "Verify" button when verified=true even if onVerify provided', () => {
    render(<VerificationBadge verified={true} onVerify={vi.fn()} />)

    expect(screen.queryByRole('button', { name: 'Verify' })).not.toBeInTheDocument()
    expect(screen.getByText('Verified')).toBeInTheDocument()
  })
})
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `cd apps/web && pnpm vitest run src/components/shared/_tests/VerificationBadge.test.tsx`
Expected: FAIL — module not found

- [ ] **Step 3: Implement the component**

Create `apps/web/src/components/shared/VerificationBadge.tsx`:

```tsx
import { CheckCircleIcon } from '@heroicons/react/16/solid'

interface VerificationBadgeProps {
  verified: boolean
  onVerify?: () => void
}

export function VerificationBadge({ verified, onVerify }: VerificationBadgeProps) {
  if (verified) {
    return (
      <span className="gap-xxs flex items-center">
        <CheckCircleIcon className="text-primary-teal h-4 w-4" />
        <span className="text-primary-teal text-small font-normal leading-[1.5] tracking-[0.21px]">
          Verified
        </span>
      </span>
    )
  }

  if (onVerify) {
    return (
      <button
        type="button"
        onClick={onVerify}
        className="text-primary-blue text-small font-medium leading-[1.5] tracking-[0.21px] hover:underline"
      >
        Verify
      </button>
    )
  }

  return null
}
```

- [ ] **Step 4: Run tests to verify they pass**

Run: `cd apps/web && pnpm vitest run src/components/shared/_tests/VerificationBadge.test.tsx`
Expected: All 5 tests PASS

- [ ] **Step 5: Commit**

```bash
git add apps/web/src/components/shared/VerificationBadge.tsx apps/web/src/components/shared/_tests/VerificationBadge.test.tsx
git commit -m "feat: add VerificationBadge component for verified/verify states"
```

---

## Task 3: Create `OtpVerificationDialog` component

**Files:**

- Create: `apps/web/src/components/shared/OtpVerificationDialog.tsx`
- Create: `apps/web/src/components/shared/_tests/OtpVerificationDialog.test.tsx`

- [ ] **Step 1: Write the failing tests**

Create `apps/web/src/components/shared/_tests/OtpVerificationDialog.test.tsx`:

```tsx
import { render, screen, waitFor } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import React from 'react'
import { beforeEach, describe, expect, it, vi } from 'vitest'

// Mock InputOTP — jsdom can't handle the real library
vi.mock('@/components/ui/input-otp', async () => {
  const ReactMod = await import('react')

  const InputOTP = ReactMod.forwardRef(
    (
      {
        maxLength,
        value,
        onChange,
        children,
        ...props
      }: {
        [key: string]: unknown
        children?: React.ReactNode
        maxLength?: number
        onChange?: (val: string) => void
        value?: string
      },
      ref: React.Ref<HTMLInputElement>,
    ) =>
      ReactMod.createElement(
        'div',
        null,
        ReactMod.createElement('input', {
          ref,
          'aria-label': 'Verification code',
          'value': value ?? '',
          maxLength,
          'onChange': (e: React.ChangeEvent<HTMLInputElement>) => onChange?.(e.target.value),
          ...props,
        }),
        children,
      ),
  )

  InputOTP.displayName = 'InputOTP'

  return {
    InputOTP,
    InputOTPGroup: ({ children }: { children: React.ReactNode }) =>
      ReactMod.createElement('div', null, children),
    InputOTPSlot: () => null,
  }
})

import { OtpVerificationDialog } from '../OtpVerificationDialog'

describe('OtpVerificationDialog', () => {
  const defaultProps = {
    open: true,
    onOpenChange: vi.fn(),
    type: 'email' as const,
    contact: 'test@example.com',
    onSendCode: vi.fn().mockResolvedValue(undefined),
    onVerifyCode: vi.fn().mockResolvedValue(true),
    onSuccess: vi.fn(),
  }

  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('renders email verification title', () => {
    render(<OtpVerificationDialog {...defaultProps} />)

    expect(screen.getByText("Verify it's you")).toBeInTheDocument()
    expect(screen.getByText('Code sent to test@example.com')).toBeInTheDocument()
  })

  it('renders phone verification description', () => {
    render(<OtpVerificationDialog {...defaultProps} type="phone" contact="5551234567" />)

    expect(screen.getByText('Code sent via SMS to 5551234567')).toBeInTheDocument()
  })

  it('calls onVerifyCode when 6-digit code is entered and Verify clicked', async () => {
    const user = userEvent.setup()
    const onVerifyCode = vi.fn().mockResolvedValue(true)

    render(<OtpVerificationDialog {...defaultProps} onVerifyCode={onVerifyCode} />)

    const input = screen.getByLabelText('Verification code')
    await user.type(input, '424242')
    await user.click(screen.getByRole('button', { name: 'Verify' }))

    await waitFor(() => {
      expect(onVerifyCode).toHaveBeenCalledWith('424242')
    })
  })

  it('calls onSuccess after successful verification', async () => {
    const user = userEvent.setup()
    const onSuccess = vi.fn()
    const onVerifyCode = vi.fn().mockResolvedValue(true)

    render(
      <OtpVerificationDialog {...defaultProps} onVerifyCode={onVerifyCode} onSuccess={onSuccess} />,
    )

    const input = screen.getByLabelText('Verification code')
    await user.type(input, '424242')
    await user.click(screen.getByRole('button', { name: 'Verify' }))

    await waitFor(() => {
      expect(onSuccess).toHaveBeenCalledTimes(1)
    })
  })

  it('shows error state when verification fails', async () => {
    const user = userEvent.setup()
    const onVerifyCode = vi.fn().mockResolvedValue(false)

    render(<OtpVerificationDialog {...defaultProps} onVerifyCode={onVerifyCode} />)

    const input = screen.getByLabelText('Verification code')
    await user.type(input, '000000')
    await user.click(screen.getByRole('button', { name: 'Verify' }))

    await waitFor(() => {
      expect(screen.getByText('Invalid code. Please try again.')).toBeInTheDocument()
    })
  })

  it('calls onSendCode when resend is clicked', async () => {
    const user = userEvent.setup()
    const onSendCode = vi.fn().mockResolvedValue(undefined)

    render(<OtpVerificationDialog {...defaultProps} onSendCode={onSendCode} />)

    const resendButton = screen.getByText('Send Again')
    await user.click(resendButton)

    await waitFor(() => {
      expect(onSendCode).toHaveBeenCalledTimes(1)
    })
  })

  it('disables Verify button when code length < 6', () => {
    render(<OtpVerificationDialog {...defaultProps} />)

    const verifyButton = screen.getByRole('button', { name: 'Verify' })

    expect(verifyButton).toBeDisabled()
  })

  it('does not render when open=false', () => {
    render(<OtpVerificationDialog {...defaultProps} open={false} />)

    expect(screen.queryByText("Verify it's you")).not.toBeInTheDocument()
  })
})
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `cd apps/web && pnpm vitest run src/components/shared/_tests/OtpVerificationDialog.test.tsx`
Expected: FAIL — module not found

- [ ] **Step 3: Implement the component**

Create `apps/web/src/components/shared/OtpVerificationDialog.tsx`. This is extracted from the existing OTP dialog in `VendorProfileForm.tsx` (lines 629-695):

```tsx
'use client'

import { Check, Loader2 } from 'lucide-react'
import { useState } from 'react'

import { BulButton } from '@/components/shared/buttons/BulButton'
import { Dialog, DialogContent, DialogHeader } from '@/components/ui/dialog'
import { InputOTP, InputOTPGroup, InputOTPSlot } from '@/components/ui/input-otp'

export interface OtpVerificationDialogProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  type: 'email' | 'phone'
  contact: string
  onSendCode: () => Promise<void>
  onVerifyCode: (code: string) => Promise<boolean>
  onSuccess: () => void
}

export function OtpVerificationDialog({
  open,
  onOpenChange,
  type,
  contact,
  onSendCode,
  onVerifyCode,
  onSuccess,
}: OtpVerificationDialogProps) {
  const [otpValue, setOtpValue] = useState('')
  const [otpError, setOtpError] = useState(false)
  const [isVerifying, setIsVerifying] = useState(false)
  const [isResending, setIsResending] = useState(false)
  const [resendSuccess, setResendSuccess] = useState(false)

  const handleVerify = async () => {
    setIsVerifying(true)
    try {
      const success = await onVerifyCode(otpValue)

      if (success) {
        setOtpValue('')
        setOtpError(false)
        onSuccess()
      } else {
        setOtpError(true)
      }
    } catch {
      setOtpError(true)
    } finally {
      setIsVerifying(false)
    }
  }

  const handleResend = async () => {
    setIsResending(true)
    setOtpError(false)
    setOtpValue('')
    setResendSuccess(false)

    try {
      await onSendCode()
      setResendSuccess(true)
      setTimeout(() => setResendSuccess(false), 3000)
    } catch {
      setOtpError(true)
    } finally {
      setIsResending(false)
    }
  }

  const handleOpenChange = (nextOpen: boolean) => {
    if (!nextOpen) {
      setOtpValue('')
      setOtpError(false)
      setResendSuccess(false)
    }

    onOpenChange(nextOpen)
  }

  return (
    <Dialog open={open} onOpenChange={handleOpenChange}>
      <DialogContent className="sm:max-w-md" hiddenTitle="Verify Identity">
        <DialogHeader>
          <h2 className="text-primary-blue text-center text-xl font-semibold">
            Verify it&apos;s you
          </h2>
          <p className="text-neutral-gray-darker text-small text-center">
            {type === 'email' ? `Code sent to ${contact}` : `Code sent via SMS to ${contact}`}
          </p>
        </DialogHeader>

        <div className="gap-lg py-md flex flex-col items-center">
          <InputOTP
            maxLength={6}
            value={otpValue}
            onChange={(value) => {
              setOtpValue(value)
              setOtpError(false)
            }}
          >
            <InputOTPGroup>
              <InputOTPSlot index={0} className="h-12 w-12 text-lg" />
              <InputOTPSlot index={1} className="h-12 w-12 text-lg" />
              <InputOTPSlot index={2} className="h-12 w-12 text-lg" />
              <InputOTPSlot index={3} className="h-12 w-12 text-lg" />
              <InputOTPSlot index={4} className="h-12 w-12 text-lg" />
              <InputOTPSlot index={5} className="h-12 w-12 text-lg" />
            </InputOTPGroup>
          </InputOTP>

          {otpError && <p className="text-small text-red-500">Invalid code. Please try again.</p>}

          <BulButton
            onClick={handleVerify}
            disabled={otpValue.length !== 6 || isVerifying}
            className="w-full"
          >
            {isVerifying ? 'Verifying...' : 'Verify'}
          </BulButton>

          <button
            onClick={handleResend}
            disabled={isResending}
            className="text-neutral-gray-darker text-small hover:text-primary-blue flex cursor-pointer items-center gap-2 disabled:cursor-not-allowed disabled:opacity-50"
          >
            {isResending ? (
              <>
                <Loader2 className="h-4 w-4 animate-spin" />
                Sending code...
              </>
            ) : resendSuccess ? (
              <>
                <Check className="h-4 w-4 text-emerald-600" />
                <span className="text-emerald-600">Code sent!</span>
              </>
            ) : (
              <>
                Didn&apos;t receive code? <span className="font-medium">Send Again</span>
              </>
            )}
          </button>
        </div>
      </DialogContent>
    </Dialog>
  )
}
```

- [ ] **Step 4: Run tests to verify they pass**

Run: `cd apps/web && pnpm vitest run src/components/shared/_tests/OtpVerificationDialog.test.tsx`
Expected: All 7 tests PASS

- [ ] **Step 5: Commit**

```bash
git add apps/web/src/components/shared/OtpVerificationDialog.tsx apps/web/src/components/shared/_tests/OtpVerificationDialog.test.tsx
git commit -m "feat: add reusable OtpVerificationDialog extracted from VendorProfileForm"
```

---

## Task 4: Create shared email verification utilities

**Files:**

- Create: `apps/web/src/lib/email-verification/emailVerificationUtils.ts`
- Create: `apps/web/src/lib/email-verification/_tests/emailVerificationUtils.test.ts`
- Modify: `apps/web/src/actions/vendors/sendVendorEmailVerificationAction.ts`
- Modify: `apps/web/src/actions/vendors/verifyVendorEmailCodeAction.ts`
- Modify: `apps/web/src/actions/members/settings/sendEmailVerificationAction.ts`
- Modify: `apps/web/src/actions/members/settings/verifyEmailCodeAction.ts`

- [ ] **Step 1: Write the failing tests for shared utils**

Create `apps/web/src/lib/email-verification/_tests/emailVerificationUtils.test.ts`:

```ts
import { describe, expect, it, vi } from 'vitest'

vi.mock('@sendgrid/mail', () => ({
  default: {
    setApiKey: vi.fn(),
    send: vi.fn().mockResolvedValue(undefined),
  },
}))

import { generateOTP } from '../emailVerificationUtils'

describe('emailVerificationUtils', () => {
  describe('generateOTP', () => {
    it('returns "424242" for test emails', () => {
      expect(generateOTP('user+test@example.com')).toBe('424242')
    })

    it('returns a 6-digit string for non-test emails', () => {
      const otp = generateOTP('user@example.com')

      expect(otp).toHaveLength(6)
      expect(Number(otp)).toBeGreaterThanOrEqual(100000)
      expect(Number(otp)).toBeLessThanOrEqual(999999)
    })
  })
})
```

- [ ] **Step 2: Run tests to verify they fail**

Run: `cd apps/web && pnpm vitest run src/lib/email-verification/_tests/emailVerificationUtils.test.ts`
Expected: FAIL — module not found

- [ ] **Step 3: Implement shared utilities**

Create `apps/web/src/lib/email-verification/emailVerificationUtils.ts`:

```ts
import sendgridMail from '@sendgrid/mail'

export function generateOTP(email: string): string {
  const isTestEmail = email.includes('+test')

  if (isTestEmail) return '424242'

  return Math.floor(100000 + Math.random() * 900000).toString()
}

export async function sendVerificationEmail({
  toEmail,
  code,
  firstName,
}: {
  toEmail: string
  code: string
  firstName: string | null
}) {
  if (!process.env.SENDGRID_API_KEY) {
    throw new Error('SENDGRID_API_KEY is not configured')
  }

  const fromEmail = process.env.SENDGRID_FROM_EMAIL

  if (!fromEmail) {
    throw new Error('SENDGRID_FROM_EMAIL is not configured')
  }

  // Skip sending for test emails — code is hardcoded "424242"
  if (toEmail.includes('+test')) return

  sendgridMail.setApiKey(process.env.SENDGRID_API_KEY)

  await sendgridMail.send({
    from: {
      name: 'Bulqit',
      email: fromEmail,
    },
    to: toEmail,
    subject: 'Verify your email address',
    html: `
      <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
        <h2 style="color: #0d9488;">Verify your email address</h2>
        <p>Hi ${firstName || 'there'},</p>
        <p>You requested to change your email address. Please use the following code to verify your new email:</p>
        <div style="background-color: #f3f4f6; padding: 20px; text-align: center; margin: 20px 0; border-radius: 8px;">
          <span style="font-size: 32px; font-weight: bold; letter-spacing: 8px; color: #111827;">${code}</span>
        </div>
        <p>This code will expire in 10 minutes.</p>
        <p>If you didn't request this change, you can safely ignore this email.</p>
        <p style="color: #6b7280; margin-top: 30px;">— The Bulqit Team</p>
      </div>
    `,
  })
}
```

- [ ] **Step 4: Run tests to verify they pass**

Run: `cd apps/web && pnpm vitest run src/lib/email-verification/_tests/emailVerificationUtils.test.ts`
Expected: All tests PASS

- [ ] **Step 5: Refactor vendor send action to use shared utils**

Replace the content of `apps/web/src/actions/vendors/sendVendorEmailVerificationAction.ts`:

```ts
'use server'

import { eq } from '@bulqit/database/orm'
import { vendors } from '@bulqit/database/schemas'

import { db } from '@/db'
import { generateOTP, sendVerificationEmail } from '@/lib/email-verification/emailVerificationUtils'
import { getCurrentVendorTeamMember } from '@/utils/vendor-team-members'

export interface SendVendorEmailVerificationInput {
  newEmail: string
}

export async function sendVendorEmailVerification(data: SendVendorEmailVerificationInput) {
  const { vendor, teamMember } = await getCurrentVendorTeamMember()

  if ('id' in teamMember && teamMember.id) {
    throw new Error('Email verification via OTP is not yet supported for team members.')
  }

  if (!vendor.id) {
    throw new Error('Vendor not found')
  }

  const code = generateOTP(data.newEmail)

  if (process.env.NODE_ENV === 'development') {
    console.log(`[DEV] Vendor email verification code for ${data.newEmail}: ${code}`)
  }

  await db
    .update(vendors)
    .set({
      emailVerificationCode: code,
      emailVerificationStatus: 'NOT_VERIFIED',
      updatedAt: new Date(),
    })
    .where(eq(vendors.id, vendor.id))

  await sendVerificationEmail({
    toEmail: data.newEmail,
    code,
    firstName: vendor.firstName,
  })

  return { success: true }
}
```

- [ ] **Step 6: Refactor vendor verify action to use shared utils**

The `verifyVendorEmailCodeAction.ts` stays mostly the same — it doesn't duplicate OTP/SendGrid logic, just validates the code. No changes needed here beyond confirming it still works.

- [ ] **Step 7: Refactor member send action to use shared utils**

Replace the content of `apps/web/src/actions/members/settings/sendEmailVerificationAction.ts`:

```ts
'use server'

import { eq } from '@bulqit/database/orm'
import { homeowners } from '@bulqit/database/schemas'

import { db } from '@/db'
import { generateOTP, sendVerificationEmail } from '@/lib/email-verification/emailVerificationUtils'
import { getCurrentMember, withMemberAuth } from '@/utils/member'

export interface SendEmailVerificationInput {
  newEmail: string
}

async function sendEmailVerification(data: SendEmailVerificationInput) {
  const member = await getCurrentMember()

  const code = generateOTP(data.newEmail)

  if (process.env.NODE_ENV === 'development') {
    console.log(`[DEV] Email verification code for ${data.newEmail}: ${code}`)
  }

  await db
    .update(homeowners)
    .set({
      emailVerificationCode: code,
      emailVerificationStatus: 'NOT_VERIFIED',
      updatedAt: new Date(),
    })
    .where(eq(homeowners.id, member.id))

  await sendVerificationEmail({
    toEmail: data.newEmail,
    code,
    firstName: member.firstName,
  })

  return { success: true }
}

export default withMemberAuth(sendEmailVerification)
```

- [ ] **Step 8: Run existing tests to verify no regressions**

Run: `cd apps/web && pnpm vitest run src/lib/email-verification/_tests/emailVerificationUtils.test.ts`
Expected: PASS

- [ ] **Step 9: Commit**

```bash
git add apps/web/src/lib/email-verification/emailVerificationUtils.ts apps/web/src/lib/email-verification/_tests/emailVerificationUtils.test.ts apps/web/src/actions/vendors/sendVendorEmailVerificationAction.ts apps/web/src/actions/members/settings/sendEmailVerificationAction.ts
git commit -m "refactor: extract shared email verification utilities from vendor/member actions"
```

---

## Task 5: Update VendorProfileForm to use shared components

**Files:**

- Modify: `apps/web/src/components/vendors/dashboard/settings/VendorProfileForm.tsx`
- Modify: `apps/web/src/app/dashboard/vendor/settings/profile/page.tsx`

- [ ] **Step 1: Remove `isEmailVerified`/`isPhoneVerified` props from the server page**

In `apps/web/src/app/dashboard/vendor/settings/profile/page.tsx`, remove lines 62-67 (verification prop computation) and the corresponding props passed to `VendorProfileForm`:

Replace the return block and remove verification logic. The page should no longer compute or pass `isEmailVerified` or `isPhoneVerified`:

```tsx
// Remove these lines (62-67):
// const isEmailVerified = isTeamMember ? true : vendor.emailVerificationStatus === 'VALID'
// const phone = isTeamMember ? teamMember.phone : vendor.phone
// const isPhoneVerified = !!phone

// Update the return to remove verification props:
return <VendorProfileForm initial={initial} imageUrl={imageUrl} role={role} location={location} />
```

- [ ] **Step 2: Update VendorProfileForm types and imports**

In `apps/web/src/components/vendors/dashboard/settings/VendorProfileForm.tsx`:

Remove `isEmailVerified` and `isPhoneVerified` from the props type (lines 35-42):

```tsx
type VendorProfileFormProps = {
  imageUrl: string | null
  initial: VendorProfileInput
  location: string | null
  role: string
}
```

Update the component signature (lines 44-51):

```tsx
export function VendorProfileForm({
  initial,
  imageUrl,
  role,
  location,
}: VendorProfileFormProps) {
```

Add imports for the new shared components. Replace the `CheckCircleIcon` import usage (line 2) — it will now come from `VerificationBadge`. Add these imports:

```tsx
import { OtpVerificationDialog } from '@/components/shared/OtpVerificationDialog'
import { VerificationBadge } from '@/components/shared/VerificationBadge'
import { useVerificationStatus } from '@/hooks/useVerificationStatus'
```

Remove import of `CheckCircleIcon` from `@heroicons/react/16/solid` (only if not used elsewhere in the file — check first; the `ExclamationTriangleIcon` is still used for the warning banner).

- [ ] **Step 3: Add `useVerificationStatus` hook call**

After the existing `usePhoneVerification()` call (around line 94), add:

```tsx
const { isEmailVerified, isPhoneVerified } = useVerificationStatus()
```

- [ ] **Step 4: Replace inline phone badge with `VerificationBadge`**

Replace the phone label section (lines 551-564) — the `<div className="flex items-center justify-between">` block:

```tsx
<div className="flex items-center justify-between">
  <label className="text-primary-blue text-small font-medium tracking-[0.35px]">
    Mobile Phone Number
  </label>
  {form.phone && (
    <VerificationBadge
      verified={isPhoneVerified}
      onVerify={() => {
        setPendingChangeType('phone')
        proceedWithVerification('phone')
      }}
    />
  )}
</div>
```

- [ ] **Step 5: Add `VerificationBadge` to email label**

Replace the email label (line 540) with a wrapper div:

```tsx
<div className="flex items-center justify-between">
  <label className="text-primary-blue text-small font-medium tracking-[0.35px]">
    Email Address
  </label>
  {form.email && (
    <VerificationBadge
      verified={isEmailVerified}
      onVerify={() => {
        setPendingChangeType('email')
        proceedWithVerification('email')
      }}
    />
  )}
</div>
```

- [ ] **Step 6: Replace inline OTP Dialog with `OtpVerificationDialog`**

Replace the entire OTP Dialog block (lines 629-695) with:

```tsx
<OtpVerificationDialog
  open={showOTPDialog}
  onOpenChange={handleCloseOTPDialog}
  type={otpType}
  contact={pendingContact}
  onSendCode={async () => {
    if (otpType === 'email') {
      await sendVendorEmailVerification({ newEmail: pendingContact })
    } else if (phoneNumberId) {
      await resendPhoneCode(phoneNumberId)
    }
  }}
  onVerifyCode={async (code) => {
    if (otpType === 'phone') {
      if (!phoneNumberId) return false
      const result = await verifyPhoneCode(phoneNumberId, code)
      return result.success
    } else {
      const result = await verifyVendorEmailCode({ code, newEmail: pendingContact })
      if (result.success && 'warning' in result && result.warning) {
        toast({
          title: 'Email partially updated',
          description: result.warning,
          variant: 'destructive',
        })
      }
      return result.success
    }
  }}
  onSuccess={() => {
    otpVerifiedRef.current = true
    advanceToNextChange(otpType)
  }}
/>
```

- [ ] **Step 7: Remove now-unused OTP state variables**

Remove these state variables (lines 73-80) since they're now handled inside `OtpVerificationDialog`:

```tsx
// Remove:
// const [otpValue, setOtpValue] = useState('')
// const [otpError, setOtpError] = useState(false)
// const [isResending, setIsResending] = useState(false)
// const [resendSuccess, setResendSuccess] = useState(false)
// const [isVerifying, setIsVerifying] = useState(false)
```

Remove the `handleVerifyOTP` function (lines 255-313) and `handleResendOTP` function (lines 315-350) — this logic is now inside `OtpVerificationDialog`.

Simplify `handleCloseOTPDialog` — remove OTP value/error cleanup (now handled in dialog):

```tsx
const handleCloseOTPDialog = (open: boolean) => {
  if (!open) {
    if (otpVerifiedRef.current) {
      otpVerifiedRef.current = false
    } else {
      if (otpType === 'email') {
        setForm((p) => ({ ...p, email: originalEmail }))
      } else {
        setForm((p) => ({ ...p, phone: originalPhone }))
        if (phoneNumberId) {
          cleanupUnverifiedPhone(phoneNumberId)
          setPhoneNumberId(null)
        }
      }
      pendingChangesRef.current = []
      setPendingChangeType(null)
    }
  }
  setShowOTPDialog(open)
}
```

Also remove unused imports: `InputOTP`, `InputOTPGroup`, `InputOTPSlot` from `@/components/ui/input-otp`, and `Check`, `Loader2` from `lucide-react` (if not used elsewhere — `Loader2` is still used for image upload).

- [ ] **Step 8: Run existing vendor tests to check for regressions**

Run: `cd apps/web && pnpm vitest run src/components/vendors/dashboard/settings/_tests/VendorProfileForm.phoneVerification.test.tsx`

Tests may need updates — see Task 7.

- [ ] **Step 9: Commit**

```bash
git add apps/web/src/components/vendors/dashboard/settings/VendorProfileForm.tsx apps/web/src/app/dashboard/vendor/settings/profile/page.tsx
git commit -m "feat: vendor profile uses shared VerificationBadge, OtpVerificationDialog, and Clerk-based status"
```

---

## Task 6: Update MemberAccountForm to use shared components

**Files:**

- Modify: `apps/web/src/components/members/settings/MemberAccountForm.tsx`

This task mirrors Task 5 for the member form. The member form is much larger (~1200 lines) so changes are targeted.

- [ ] **Step 1: Add imports for shared components**

Add to the imports section of `MemberAccountForm.tsx`:

```tsx
import { OtpVerificationDialog } from '@/components/shared/OtpVerificationDialog'
import { VerificationBadge } from '@/components/shared/VerificationBadge'
import { useVerificationStatus } from '@/hooks/useVerificationStatus'
```

- [ ] **Step 2: Add `useVerificationStatus` hook call**

Inside the component, add after existing hooks:

```tsx
const { isEmailVerified, isPhoneVerified } = useVerificationStatus()
```

- [ ] **Step 3: Add `VerificationBadge` to email field label**

Find the email field label in the form and wrap it:

```tsx
<div className="flex items-center justify-between">
  <label className="text-primary-blue text-small font-medium tracking-[0.35px]">
    Email Address
  </label>
  {initialData.email && (
    <VerificationBadge
      verified={isEmailVerified}
      onVerify={() => {
        setPendingChangeType('email')
        proceedWithVerification('email')
      }}
    />
  )}
</div>
```

- [ ] **Step 4: Add `VerificationBadge` to phone field label**

Find the phone field label and wrap it similarly:

```tsx
<div className="flex items-center justify-between">
  <label className="text-primary-blue text-small font-medium tracking-[0.35px]">
    Mobile Phone Number
  </label>
  {initialData.phone && (
    <VerificationBadge
      verified={isPhoneVerified}
      onVerify={() => {
        setPendingChangeType('phone')
        proceedWithVerification('phone')
      }}
    />
  )}
</div>
```

- [ ] **Step 5: Remove yellow warning banners for email/phone**

Remove the warning banner blocks that show when email or phone are missing (e.g., `{!initialData.email && (<div className="mb-4 flex items-center ...">...`)). These are replaced by the inline `VerificationBadge`.

- [ ] **Step 6: Replace inline OTP Dialog with `OtpVerificationDialog`**

Replace the member form's inline OTP dialog (similar structure to vendor, around line 1573) with `OtpVerificationDialog`, using the same pattern as Task 5 Step 6 but with member-specific callbacks:

```tsx
<OtpVerificationDialog
  open={showOTPDialog}
  onOpenChange={handleCloseOTPDialog}
  type={otpType}
  contact={pendingContact}
  onSendCode={async () => {
    if (otpType === 'email') {
      await sendEmailVerification({ newEmail: pendingContact })
    } else if (phoneNumberId) {
      await resendPhoneCode(phoneNumberId)
    }
  }}
  onVerifyCode={async (code) => {
    if (otpType === 'phone') {
      if (!phoneNumberId) return false
      const result = await verifyPhoneCode(phoneNumberId, code)
      return result.success
    } else {
      const result = await verifyEmailCode({ code, newEmail: pendingContact })
      if (result.success && 'warning' in result && result.warning) {
        toast({
          title: 'Email partially updated',
          description: result.warning,
          variant: 'destructive',
        })
      }
      return result.success
    }
  }}
  onSuccess={() => {
    otpVerifiedRef.current = true
    advanceToNextChange(otpType)
  }}
/>
```

- [ ] **Step 7: Remove now-unused OTP state and handlers**

Same cleanup as Task 5 Step 7 — remove `otpValue`, `otpError`, `isResending`, `resendSuccess`, `isVerifying` state, and `handleVerifyOTP`/`handleResendOTP` functions. Remove unused `InputOTP`-related imports.

- [ ] **Step 8: Commit**

```bash
git add apps/web/src/components/members/settings/MemberAccountForm.tsx
git commit -m "feat: member account uses shared VerificationBadge, OtpVerificationDialog, and Clerk-based status"
```

---

## Task 7: Update existing tests

**Files:**

- Modify: `apps/web/src/components/vendors/dashboard/settings/_tests/VendorProfileForm.phoneVerification.test.tsx`
- Modify: `apps/web/src/components/members/settings/_tests/MemberAccountForm.phoneVerification.test.tsx`

- [ ] **Step 1: Add `useVerificationStatus` mock to vendor tests**

In `VendorProfileForm.phoneVerification.test.tsx`, add the mock after existing mocks:

```tsx
const mockIsEmailVerified = false
const mockIsPhoneVerified = false

vi.mock('@/hooks/useVerificationStatus', () => ({
  useVerificationStatus: () => ({
    isEmailVerified: mockIsEmailVerified,
    isPhoneVerified: mockIsPhoneVerified,
  }),
}))
```

Update the component render calls to remove `isEmailVerified` and `isPhoneVerified` props — they're no longer part of the component API.

- [ ] **Step 2: Update vendor test renders**

Find all `render(<VendorProfileForm ... />)` calls and remove the `isEmailVerified` and `isPhoneVerified` props. For example:

```tsx
// Before:
render(
  <VendorProfileForm
    initial={defaultInitial}
    imageUrl={null}
    role="Owner"
    location={null}
    isEmailVerified={true}
    isPhoneVerified={true}
  />,
)

// After:
render(<VendorProfileForm initial={defaultInitial} imageUrl={null} role="Owner" location={null} />)
```

- [ ] **Step 3: Add `useVerificationStatus` mock to member tests**

Same pattern as Step 1 but in `MemberAccountForm.phoneVerification.test.tsx`.

- [ ] **Step 4: Run all affected tests**

Run: `cd apps/web && pnpm vitest run src/components/vendors/dashboard/settings/_tests/ src/components/members/settings/_tests/ src/components/shared/_tests/ src/hooks/_tests/`
Expected: All tests PASS

- [ ] **Step 5: Commit**

```bash
git add apps/web/src/components/vendors/dashboard/settings/_tests/VendorProfileForm.phoneVerification.test.tsx apps/web/src/components/members/settings/_tests/MemberAccountForm.phoneVerification.test.tsx
git commit -m "test: update vendor and member form tests for shared verification components"
```

---

## Task 8: Type check and final verification

- [ ] **Step 1: Run type checking**

Run: `cd /Users/alissongalindo/Documents/GitHub/FW/bul-website && pnpm check-types`
Expected: No type errors

- [ ] **Step 2: Run full test suite**

Run: `cd apps/web && pnpm test:unit`
Expected: All tests PASS

- [ ] **Step 3: Run linting**

Run: `pnpm lint`
Expected: No lint errors

- [ ] **Step 4: Fix any issues found in Steps 1-3**

If any type errors, test failures, or lint issues are found, fix them before proceeding.

- [ ] **Step 5: Final commit (if any fixes were needed)**

```bash
git add -A
git commit -m "fix: resolve type/lint issues from verification labels refactor"
```
