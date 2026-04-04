# API routes (`apps/web/src/app/api`)

Next.js **Route Handlers** (`route.ts`) implement REST-ish endpoints. 95 route handlers on staging. Read each `route.ts` for full detail.

## Health and diagnostics

| Path | Synopsis |
|------|----------|
| `api/health/route.ts` | `checkServices()` per-service status, overall healthy/unhealthy, HTTP 200 vs 503 |
| `api/healthz/route.ts` | Minimal JSON `{ ok, timestamp, version }` using `VERCEL_GIT_COMMIT_SHA` |
| `api/admin/db/diagnostics/route.ts` | Admin-only DB diagnostics |
| `api/rate-limit/route.ts` | Rate-limit status check |

## CMS

| Path | Synopsis |
|------|----------|
| `api/draft-mode/enable/route.ts` | `defineEnableDraftMode` from `next-sanity/draft-mode` for Sanity preview |
| `api/sanity/revalidate/route.ts` | On-demand revalidation triggered by Sanity webhooks |

## Dashboard

| Path | Synopsis |
|------|----------|
| `api/dashboard/alerts/route.ts` | List dashboard alerts |
| `api/dashboard/alerts/[id]/route.ts` | Single alert detail |
| `api/dashboard/alerts/[id]/read/route.ts` | Mark alert as read |
| `api/dashboard/member/jobs/rate/route.ts` | Member job rating |

## Notifications

| Path | Synopsis |
|------|----------|
| `api/notifications/events/route.ts` | Notification event ingestion |

## Member documents

| Path | Synopsis |
|------|----------|
| `api/member/invoices/[id]/pdf/route.ts` | Member invoice PDF generation |

## Vendor routes

| Path | Synopsis |
|------|----------|
| `api/vendor/me/route.ts` | Current vendor profile |
| `api/vendor/transactions/[id]/invoice/pdf/route.ts` | Vendor invoice PDF |
| `api/vendor/transactions/[id]/invoice/stripe/route.ts` | Stripe invoice lookup |
| `api/vendor/disbursements/[id]/receipt/pdf/route.ts` | Payout disbursement receipt |
| `api/vendor/bidding/block-report/pdf/route.ts` | Block report PDF |
| `api/vendor/export-jobs/route.ts` | Export vendor jobs |
| `api/vendor/export-member-jobs/route.ts` | Export member jobs |
| `api/vendor/export-upcoming-jobs-by-date/route.ts` | Export upcoming jobs by date |

## Messaging (Twilio)

| Path | Synopsis |
|------|----------|
| `api/messaging/token/route.ts` | Twilio access token for client SDK |
| `api/messaging/conversations/route.ts` | List/create conversations |
| `api/messaging/conversations/[conversationSid]/messages/route.ts` | List/send messages |
| `api/messaging/conversations/[conversationSid]/participants/route.ts` | Manage participants |
| `api/messaging/conversations/[conversationSid]/participants/sms/route.ts` | SMS participant management |
| `api/messaging/conversations/[conversationSid]/read/route.ts` | Mark conversation read |
| `api/messaging/conversations/[conversationSid]/join/route.ts` | Join a conversation |
| `api/messaging/conversations/[conversationSid]/close/route.ts` | Close a conversation |
| `api/messaging/conversations/[conversationSid]/export/route.ts` | Export conversation |
| `api/messaging/conversations/[conversationSid]/identities/route.ts` | Participant identities |
| `api/messaging/conversations/[conversationSid]/media/route.ts` | Media upload |
| `api/messaging/conversations/[conversationSid]/messages/[messageSid]/media/[mediaSid]/route.ts` | Media retrieval |
| `api/messaging/conversations/[conversationSid]/call-proxy/route.ts` | Voice call proxy |
| `api/messaging/conversations/link/route.ts` | Link conversation to entity |
| `api/messaging/user/conversations/route.ts` | User's conversation list (large endpoint, ~1,325 lines) |
| `api/messaging/relationships/route.ts` | Messaging relationships |
| `api/messaging/support-conversations/route.ts` | Support conversation management |
| `api/messaging/vendors/[vendorId]/route.ts` | Vendor messaging context |
| `api/messaging/flex/config/route.ts` | Twilio Flex configuration |
| `api/messaging/flex/conversation/[conversationSid]/route.ts` | Flex conversation bridge |

## Twilio webhooks

| Path | Synopsis |
|------|----------|
| `api/twilio/conversations/webhook/route.ts` | Conversation event webhooks (signature verified) |
| `api/twilio/voice/proxy/route.ts` | Voice proxy callbacks |

## Admin messaging

| Path | Synopsis |
|------|----------|
| `api/admin/messaging/conversations/route.ts` | Admin conversation list |
| `api/admin/messaging/conversations/[conversationSid]/join/route.ts` | Admin join conversation |
| `api/admin/messaging/conversations/[conversationSid]/participants/route.ts` | Admin participant management |
| `api/admin/messaging/conversations/[conversationSid]/messages/[messageSid]/media/[mediaSid]/route.ts` | Admin media retrieval |
| `api/admin/messaging/conversations/[conversationSid]/simulate-inbound-sms/route.ts` | SMS simulation for testing |
| `api/admin/messaging/phone-pool/route.ts` | Phone pool management |
| `api/admin/messaging/phone-pool/health/route.ts` | Phone pool health check |
| `api/admin/messaging/phone-pool/sync/route.ts` | Phone pool sync with Twilio |
| `api/admin/messaging/relationships/route.ts` | Admin messaging relationships |
| `api/admin/messaging/send-attempts/route.ts` | Send attempt history |
| `api/admin/messaging/setup-test/route.ts` | Test setup |
| `api/admin/messaging/sms-readiness/route.ts` | SMS readiness check |
| `api/admin/messaging/support-conversations/route.ts` | Support conversation admin |
| `api/admin/messaging/support-targets/route.ts` | Support targets |
| `api/admin/messaging/vendor-contact-numbers/route.ts` | Vendor contact number management |
| `api/admin/messaging/voice-readiness/route.ts` | Voice readiness check |
| `api/admin/dev/messaging/campaign-events/route.ts` | Dev campaign event simulation |

## Admin service opportunities

| Path | Synopsis |
|------|----------|
| `api/admin/service-opportunities/route.ts` | List service opportunities |
| `api/admin/service-opportunities/diagnostics/route.ts` | Diagnostics |
| `api/admin/service-opportunities/create-test/route.ts` | Create test opportunity |
| `api/admin/service-opportunities/setup-test-interest/route.ts` | Set up test interest data |
| `api/admin/service-opportunities/[id]/messaging/provision/route.ts` | Provision messaging for opportunity |
| `api/admin/service-opportunities/[id]/messaging/close/route.ts` | Close messaging for opportunity |
| `api/admin/service-opportunities/[id]/messaging/sync/route.ts` | Sync messaging state |
| `api/admin/service-opportunities/[id]/setup-test-data/route.ts` | Set up test data |
| `api/admin/service-opportunities/[id]/stage/route.ts` | Stage opportunity |
| `api/admin/service-opportunities/[id]/vendor/route.ts` | Assign vendor |

## Admin batch / import / export

| Path | Synopsis |
|------|----------|
| `api/admin/export-blocks/route.ts` | Export blocks CSV |
| `api/admin/export-jobs/route.ts` | Export jobs CSV |
| `api/admin/export-lots/route.ts` | Export lots CSV |
| `api/admin/export-members/route.ts` | Export members CSV |
| `api/admin/export-referral-campaigns/route.ts` | Export referral campaigns CSV |
| `api/admin/export-services/route.ts` | Export services CSV |
| `api/admin/export-transactions/route.ts` | Export transactions CSV |
| `api/admin/export-vendors/route.ts` | Export vendors CSV |
| `api/admin/import-services/route.ts` | Import services |
| `api/admin/upload-lots/route.ts` | Bulk lot upload |
| `api/admin/upload-members/route.ts` | Bulk member upload |
| `api/admin/upload-vendors/route.ts` | Bulk vendor upload |
| `api/admin/lots/bulk/route.ts` | Bulk lot operations |
| `api/admin/add-ons/[addOnId]/status/route.ts` | Add-on status update |

## Admin payouts

| Path | Synopsis |
|------|----------|
| `api/admin/payouts/run-batch/route.ts` | Trigger batch payout |
| `api/admin/payouts/run-batch/status/route.ts` | Poll batch payout status |
| `api/admin/payouts/[id]/pdf/route.ts` | Payout PDF |
| `api/admin/payouts/export/route.ts` | Export payouts |
| `api/admin/transactions/[id]/invoice/pdf/route.ts` | Admin transaction invoice PDF |

## Webhooks

| Path | Synopsis |
|------|----------|
| `api/webhooks/stripe/route.ts` | Stripe event processing (payments, subscriptions, payouts, dunning) |
| `api/webhooks/support-inbound-email/route.ts` | Inbound support email routing |
| `api/sendgrid/events/webhook/route.ts` | SendGrid event webhook (delivery tracking) |
| `api/sendgrid/inbound/webhook/route.ts` | SendGrid inbound email parsing |

## Cron

| Path | Synopsis |
|------|----------|
| `api/cron/sms-pool-health/route.ts` | SMS phone pool health check (Vercel cron, validates `CRON_SECRET`) |

## Examples

| Path | Synopsis |
|------|----------|
| `api/sentry-example-api/route.ts` | Sentry capture demonstration |
| `api/temporal-example-api/route.ts` | Temporal workflow test |

**Cross-cutting:** Most handlers use `NextResponse.json`, Zod or manual validation, and either Clerk session (`auth()`) or shared secret (webhooks, cron). Admin routes often wrap `withApiAdminAuth`.
