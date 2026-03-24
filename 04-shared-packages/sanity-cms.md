# `packages/sanity-cms`

## Paths

| Item | Path |
|------|------|
| Package | `packages/sanity-cms/` |
| Schemas | `packages/sanity-cms/src/schemas/` |
| Server helpers | `packages/sanity-cms/src/server.ts` (and `dist/server`) — `sanityFetch`, `pageByPathQuery`, `client`, `token` |

## Behavior synopsis

- **Schemas** — Define document types for marketing pages, singletons, and CMS structure consumed by Studio (`apps/web/sanity.config.ts` points here).
- **`server` export** — Authenticated and public **Sanity clients**, GROQ queries, and **`sanityFetch`** caching helpers used by **`[...slug]/page.tsx`** and **`api/draft-mode`**.
- **`defineEnableDraftMode`** consumers use **`client.withConfig({ token })`** for preview.

**Features controlled:** Structured content, SEO metadata on pages, and editor workflow in Studio.

**Commands:** Root **`pnpm typegen`** regenerates TS types from schemas for the app.
