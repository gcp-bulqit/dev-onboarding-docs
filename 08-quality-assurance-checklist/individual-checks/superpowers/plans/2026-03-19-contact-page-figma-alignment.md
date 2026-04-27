# Contact Us Page — Figma Alignment Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Align the Contact Us page (`/contact-us`) with the Figma design (node `4884:1631`) across both sections.

**Architecture:** Surgical edits to 4 existing components — fix padding, replace submit button, refactor card grid layouts. No new files. No schema changes.

**Tech Stack:** Next.js, React, Tailwind CSS, existing shared components (CtaButton for support buttons only)

**Ticket:** CU-868h6aza7

**Figma:** `https://www.figma.com/design/JnpQj0zPgmGs571joZJG7g/-BUL--Visual-Design---Phase-2---Vendor?node-id=4884-1631`

---

## File Map

| File                                                                       | Action | Responsibility                                                   |
| -------------------------------------------------------------------------- | ------ | ---------------------------------------------------------------- |
| `apps/web/src/app/(public)/contact-us/_components/ContactUsSection.tsx`    | Modify | Fix section paddings to match Figma                              |
| `apps/web/src/app/(public)/contact-us/_components/SmsSignupForm.tsx`       | Modify | Replace pill-shaped CtaButton with rounded-lg button             |
| `apps/web/src/app/(public)/contact-us/_components/AddressCards.tsx`        | Modify | Group title+address per card instead of separate rows            |
| `apps/web/src/app/(public)/contact-us/_components/ContactSupportCards.tsx` | Modify | Group title+description+button per card instead of separate rows |

---

### Task 1: Fix Section Paddings in ContactUsSection

**Files:**

- Modify: `apps/web/src/app/(public)/contact-us/_components/ContactUsSection.tsx`

- [ ] **Step 1: Update section 1 padding**

The current section 1 wrapper has `pt-[8.5rem]` (136px) which is too much. Figma shows `pt-0` after the navbar with `pb-6xl` (64px).

Change line 17:

```tsx
// FROM:
<div className="px-6 pb-6xl pt-[8.5rem] lg:px-7xl">
// TO:
<div className="px-6 pb-6xl pt-6xl lg:px-7xl">
```

`pt-6xl` = 64px, which provides breathing room after the navbar without the excessive 136px gap.

- [ ] **Step 2: Verify section 2 padding**

Current section 2: `px-6 py-6xl lg:px-7xl` — this maps to `py-64px px-72px` on desktop, matching Figma's `px-[72px] py-[64px]`. No change needed.

- [ ] **Step 3: Commit**

```bash
git add apps/web/src/app/(public)/contact-us/_components/ContactUsSection.tsx
git commit -m "CU-868h6aza7"
```

---

### Task 2: Replace Submit Button in SmsSignupForm

**Files:**

- Modify: `apps/web/src/app/(public)/contact-us/_components/SmsSignupForm.tsx`

- [ ] **Step 1: Replace CtaButton with styled button**

Figma shows: `bg-primary-blue`, `rounded-[8px]`, `h-[38px]`, full-width, `text-small font-medium`, white text, border `border-shades-dark-100`.

Replace the CtaButton at lines 423-429:

```tsx
// FROM:
<CtaButton
  label={(isSubmitting ? submittingButtonText : submitButtonText) || 'Contact Us'}
  type="submit"
  disabled={isSubmitting}
  fullWidth
  datatest-id="sms-signup-form-button-submit"
/>

// TO:
<button
  type="submit"
  disabled={isSubmitting}
  datatest-id="sms-signup-form-button-submit"
  className="h-[38px] w-full rounded-lg border border-shades-dark-100 bg-primary-blue font-medium leading-[1.5] tracking-[0.21px] text-neutral-white text-small hover:bg-primary-blue-dark disabled:cursor-not-allowed disabled:opacity-50"
>
  {(isSubmitting ? submittingButtonText : submitButtonText) || 'Contact Us'}
</button>
```

- [ ] **Step 2: Remove unused CtaButton import if no longer needed**

Check if `CtaButton` is still imported. Since it's only used for the submit button in this file, remove the import:

```tsx
// REMOVE this line:
import CtaButton from '@/components/shared/buttons/CtaButton'
```

- [ ] **Step 3: Commit**

```bash
git add apps/web/src/app/(public)/contact-us/_components/SmsSignupForm.tsx
git commit -m "CU-868h6aza7"
```

---

### Task 3: Refactor AddressCards Layout

**Files:**

- Modify: `apps/web/src/app/(public)/contact-us/_components/AddressCards.tsx`

- [ ] **Step 1: Refactor AddressCardRow to group title+address per card**

Current approach renders all titles first, then all addresses in a separate map. Figma groups them vertically per card.

Replace the `AddressCardRow` component:

```tsx
// FROM (lines 15-33):
const AddressCardRow: FC<AddressCardRowProps> = ({ cards, className }) => (
  <div
    className={cn('text-primary-blue grid gap-x-6 gap-y-2', className)}
    style={{ gridTemplateColumns: `repeat(${cards.length}, 1fr)` }}
  >
    {cards.map((card) => (
      <div key={`card-${card._key}-title`} className="md:max-w-[306px]">
        <div className="text-h6 font-bold leading-tight">{card.title}</div>
      </div>
    ))}
    {cards.map((card) => (
      <div key={`card-${card._key}-address`} className="md:max-w-[306px]">
        <div className="text-regular">
          <RichTextContent content={(card.address || []) as PortableTextBlock[]} />
        </div>
      </div>
    ))}
  </div>
)

// TO:
const AddressCardRow: FC<AddressCardRowProps> = ({ cards, className }) => (
  <div className={cn('text-primary-blue flex gap-6', className)}>
    {cards.map((card) => (
      <div key={card._key} className="flex flex-col gap-2 md:max-w-[306px]">
        <div className="text-h6 font-bold leading-tight">{card.title}</div>
        <div className="text-regular">
          <RichTextContent content={(card.address || []) as PortableTextBlock[]} />
        </div>
      </div>
    ))}
  </div>
)
```

Key changes:

- Single map loop instead of two separate loops
- `flex gap-6` instead of CSS grid (matches Figma's `gap-[24px]` horizontal)
- `gap-2` (8px) between title and address within each card (matches Figma's `gap-[8px]`)

- [ ] **Step 2: Commit**

```bash
git add apps/web/src/app/(public)/contact-us/_components/AddressCards.tsx
git commit -m "CU-868h6aza7"
```

---

### Task 4: Refactor ContactSupportCards Layout

**Files:**

- Modify: `apps/web/src/app/(public)/contact-us/_components/ContactSupportCards.tsx`

- [ ] **Step 1: Refactor ContactSupportCardRow to group per card**

Current approach renders titles, descriptions, and buttons in 3 separate map loops. Figma groups them per card.

Replace the `ContactSupportCardRow` component:

```tsx
// FROM (lines 17-47):
const ContactSupportCardRow: FC<ContactSupportCardRowProps> = ({ cards, rowId }) => (
  <div
    className="text-primary-blue grid gap-x-6 gap-y-2"
    style={{ gridTemplateColumns: `repeat(${cards.length}, 1fr)` }}
  >
    {cards.map((card) => (
      <div key={`row-${rowId}-card-${card._key}-title`} className="md:max-w-[306px]">
        <div className="text-h6 font-bold leading-tight">{card.title}</div>
      </div>
    ))}
    {cards.map((card) => (
      <div key={`row-${rowId}-card-${card._key}-description`} className="mb-4 md:max-w-[306px]">
        <div className="text-regular">
          <RichTextContent content={(card.description || []) as PortableTextBlock[]} />
        </div>
      </div>
    ))}
    {cards.map((card) => (
      <div key={`row-${rowId}-card-${card._key}-cta`} className="md:max-w-[306px]">
        <CtaButton
          inverted
          height={32}
          label={card.buttonTitle ?? ''}
          className="mx-0"
          fontSize="text-xxsmall"
          href={card.buttonUrl}
        />
      </div>
    ))}
  </div>
)

// TO:
const ContactSupportCardRow: FC<ContactSupportCardRowProps> = ({ cards, rowId }) => (
  <div className="text-primary-blue flex gap-6">
    {cards.map((card) => (
      <div key={`row-${rowId}-card-${card._key}`} className="flex flex-1 flex-col gap-6">
        <div className="flex flex-col gap-2 pr-8">
          <div className="text-h6 font-bold leading-tight">{card.title}</div>
          <div className="text-regular">
            <RichTextContent content={(card.description || []) as PortableTextBlock[]} />
          </div>
        </div>
        <CtaButton
          inverted
          height={32}
          label={card.buttonTitle ?? ''}
          className="mx-0"
          fontSize="text-xxsmall"
          href={card.buttonUrl}
        />
      </div>
    ))}
  </div>
)
```

Key changes:

- Single map loop grouping title + description + button per card
- `flex gap-6` for horizontal card layout (24px, matches Figma)
- `gap-6` (24px) between text block and button within each card (matches Figma's `gap-[24px]`)
- `gap-2` (8px) between title and description
- `pr-8` (32px) right padding on text block (matches Figma's `pr-[var(--2xl,32px)]`)
- `flex-1` on each card for equal width distribution

- [ ] **Step 2: Commit**

```bash
git add apps/web/src/app/(public)/contact-us/_components/ContactSupportCards.tsx
git commit -m "CU-868h6aza7"
```

---

### Task 5: Visual Verification

- [ ] **Step 1: Take Playwright screenshot and compare**

```bash
npx playwright screenshot --browser chromium --viewport-size "1440,900" --full-page http://localhost:3000/contact-us /tmp/contact-us-after.png
```

Open `/tmp/contact-us-after.png` and compare with the Figma screenshot. Check:

- Section 1: Reduced top padding, square-cornered submit button
- Section 2: Cards properly grouped vertically per card
- Both sections: Overall spacing matches Figma

- [ ] **Step 2: Check mobile responsiveness**

```bash
npx playwright screenshot --browser chromium --viewport-size "375,812" --full-page http://localhost:3000/contact-us /tmp/contact-us-mobile.png
```

Verify mobile layout still works (single column, stacked cards).
