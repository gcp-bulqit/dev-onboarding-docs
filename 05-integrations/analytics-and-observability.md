# Analytics and observability

## Environment (see root `README.md`)

| Service | Typical env vars |
|---------|------------------|
| Segment | `NEXT_PUBLIC_SEGMENT_WRITE_KEY` |
| FullStory | `NEXT_PUBLIC_FULLSTORY_ORG_ID` |
| Sentry | `SENTRY_DSN`, `NEXT_PUBLIC_SENTRY_DSN`; edge config `apps/web/sentry.edge.config.ts` |
| Userback | `NEXT_PUBLIC_USERBACK_TOKEN` |
| Google Analytics | `NEXT_PUBLIC_GOOGLE_ANALYTICS_TAG_ID` (defined in `.env.local.example`, not referenced in code) |

## Paths

| Item | Path |
|------|------|
| Sentry server config | `apps/web/sentry.server.config.ts` |
| Sentry edge config | `apps/web/sentry.edge.config.ts` |
| Sentry client instrumentation | `apps/web/src/instrumentation-client.ts` |
| Sentry server instrumentation | `apps/web/src/instrumentation.ts` |
| Global client errors | `apps/web/src/components/GlobalErrorHandler.tsx` |
| Error boundary | `apps/web/src/app/error.tsx` |
| Global error boundary | `apps/web/src/app/global-error.tsx` |
| Segment provider | `apps/web/src/components/providers/SegmentProvider.tsx` |
| Segment page tracker | `apps/web/src/components/analytics/SegmentPageTracker.tsx` |
| Twilio Segment context | `apps/web/src/contexts/twilio-segment.tsx` |
| FullStory provider | `apps/web/src/components/providers/FullstoryProvider.tsx` |
| Feedback widget | `apps/web/src/components/providers/UserbackProvider.tsx` |
| Sentry example API | `apps/web/src/app/api/sentry-example-api/route.ts` |

## Sentry

**Configuration:** Server, edge, and client instrumentation files initialize Sentry. The `next.config.ts` includes `withSentryConfig` which routes browser Sentry requests through a Next.js rewrite (tunnel) to bypass ad blockers.

**Error boundaries:** `error.tsx` and `global-error.tsx` both call `Sentry.captureException`. `GlobalErrorHandler.tsx` subscribes to `window` errors and unhandled rejections, applies allow/deny heuristics (e.g. Stripe iframe noise on staging), reports to Sentry, and may trigger `router.refresh` on certain HTTP errors.

**Other capture points:** `validateEmailWithZeroBounce.ts`, `validatePhoneNumberAction.ts`, `LoadingStep.tsx` (onboarding), `worker-notifications/index.ts` (3 capture points), `worker-zoho-sync/cycle.ts` (data validation).

**Current coverage:** Sentry capture calls (`captureException`/`captureMessage`) appear in a small number of runtime source files: error boundaries, two validation actions, the onboarding loading step, and the notification and Zoho sync workers. The majority of error handling in the codebase uses `console.error` rather than Sentry, so production errors outside the instrumented paths are only visible in the browser console or worker stdout.

## Segment

**Initialization:** `SegmentProvider.tsx` wraps the app layout and lazily loads the `@segment/analytics-next` SDK. `TwilioSegmentProvider` in `contexts/twilio-segment.tsx` provides the analytics instance to child components.

**Page tracking:** `SegmentPageTracker.tsx` calls `analytics.page()` on route changes. This is the only tracking call in the codebase. No custom `.track()` event calls exist; business event tracking has not been implemented.

**FullStory:** `FullstoryProvider.tsx` initializes FullStory for session replay using `NEXT_PUBLIC_FULLSTORY_ORG_ID`.

## Userback

Lazy-loads the widget (deferred via `setTimeout` with a 10-second delay) to reduce main-thread cost. Identifies users from Clerk when signed in.

## Google Analytics

`NEXT_PUBLIC_GOOGLE_ANALYTICS_TAG_ID` is defined in `.env.local.example` but is not referenced in any application code. Google Analytics is not currently active.

**Features controlled:** **Error capture** (Sentry, limited coverage), **session replay** (FullStory), **page view tracking** (Segment), and **user feedback** (Userback).
