# Public marketing and content

## Paths

| Feature | Code path |
|---------|-----------|
| Public route group | `apps/web/src/app/(public)/` |
| Static marketing examples | `(public)/about-us/`, `(public)/vendors/`, `(public)/contact-us/`, `(public)/how-it-works/`, `(public)/homeowners/`, `(public)/refer/`, `(public)/terms-of-service/`, etc. |
| CMS catch-all page | `apps/web/src/app/(public)/[...slug]/page.tsx` |
| Draft mode API | `apps/web/src/app/api/draft-mode/` |
| CMS rendering | `apps/web/src/components/cms/ContentPage.tsx`, `CustomPortableText`, etc. |

## Behavior synopsis

### `(public)/[...slug]/page.tsx`

- **`generateMetadata`** — Calls `sanityFetch` with `pageByPathQuery` and builds path from `slug` segments; uses `generateSEOMetadata` for title/description from Sanity `metadata`.
- **Default export `Page`** — Sets `dynamic = 'force-dynamic'`, calls `unstable_noStore()` to avoid stale CMS HTML, fetches the page document by path; **`notFound()`** if missing; otherwise passes **`title`**, **`subheadline`**, **`body`** (Portable Text) into **`ContentPage`**.

### `ContentPage.tsx`

- **`slugify`** — Normalizes heading text to `id` attributes for in-page anchors.
- **`ptComponents`** — Overrides Portable Text `h2`–`h6` blocks to inject those IDs; custom list styling.
- **`ContentPage`** — Renders the marketing-style header band (title + subheadline) and **`CustomPortableText`** for the body inside a constrained prose column.

### `api/draft-mode/enable`

- **`defineEnableDraftMode`** (next-sanity) — GET handler that sets **Sanity draft mode** cookies using the configured authenticated Sanity client so previews can show unpublished content.

**Features controlled:** SEO landing pages, legal/static pages, and **Sanity-driven** flexible pages with preview, all without redeploying code.

## Related docs

- `docs/cms-migrations.md`
- `../05-integrations/sanity.md`
