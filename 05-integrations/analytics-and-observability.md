# Analytics and observability

## Environment (see root `README.md`)

| Service | Typical env vars |
|---------|------------------|
| Segment | `NEXT_PUBLIC_SEGMENT_WRITE_KEY` |
| FullStory | `NEXT_PUBLIC_FULLSTORY_ORG_ID` |
| Sentry | `SENTRY_DSN`, `NEXT_PUBLIC_SENTRY_DSN`; edge config `apps/web/sentry.edge.config.ts` |
| Userback | `NEXT_PUBLIC_USERBACK_TOKEN` |

## Paths

| Item | Path |
|------|------|
| Global client errors | `apps/web/src/components/GlobalErrorHandler.tsx` — `Sentry.captureException`, filters noisy browser errors, optional redirects on HTTP status |
| Feedback widget | `apps/web/src/components/providers/UserbackProvider.tsx` — loads Userback script after delay; ties to Clerk user when signed in |
| Example API | `apps/web/src/app/api/sentry-example-api/route.ts` |

## Behavior synopsis

- **Sentry** — `GlobalErrorHandler` subscribes to `window` errors/unhandled rejections, applies **allow/deny** heuristics (e.g. Stripe iframe noise on staging), reports to Sentry, and may **router.refresh** on certain HTTP errors.
- **Userback** — Lazy-loads the widget (~10s) to reduce main-thread cost; identifies users from **Clerk** when available.
- **Segment / FullStory** — Usually initialized in app instrumentation or layout (search `NEXT_PUBLIC_SEGMENT` / FullStory in `apps/web/src`).

**Features controlled:** **Error visibility**, **session replay**, **product analytics**, and **user feedback** collection.
