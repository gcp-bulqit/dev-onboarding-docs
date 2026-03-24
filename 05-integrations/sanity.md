# Sanity

## Paths

| Item | Path |
|------|------|
| Package | `packages/sanity-cms/` |
| Studio | `apps/web/src/app/cms/[[...index]]/`, `Studio.tsx` |
| Public pages | `apps/web/src/app/(public)/[...slug]/page.tsx` |
| Draft API | `apps/web/src/app/api/draft-mode/enable/route.ts` |

## Behavior synopsis

- **Sanity Client + GROQ** — `sanityFetch` / `pageByPathQuery` load page documents by URL path; **`generateMetadata`** uses the same query for SEO.
- **Studio** — `NextStudio` embeds editing UI; schemas live in `packages/sanity-cms`.
- **Draft mode** — `defineEnableDraftMode` sets cookies so preview requests see unpublished drafts.

**Features controlled:** Marketing copy, flexible landing pages, and **preview before publish**.

## Related

- `docs/cms-migrations.md`
