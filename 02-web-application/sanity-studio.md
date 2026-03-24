# Sanity Studio (embedded)

## Paths

| Item | Path |
|------|------|
| Route shell | `apps/web/src/app/cms/[[...index]]/page.tsx` |
| Studio component | `apps/web/src/app/cms/[[...index]]/Studio.tsx` |
| Sanity config | `apps/web/sanity.config.ts` (imported as `@/../sanity.config` from Studio) |
| Schemas / types | `packages/sanity-cms/` |

## Behavior synopsis

### `page.tsx`

- Exports **`dynamic = 'force-static'`** for the Studio shell.
- Re-exports **`metadata`** and **`viewport`** from `next-sanity/studio` for correct Studio chrome on Next.
- Default export renders **`Studio`**.

### `Studio.tsx` (client)

- Renders **`NextStudio`** from `next-sanity/studio` with the shared **`config`** — this mounts **Sanity Studio v3** inside the Next app so editors can manage documents without a separate host.

### Commands

- Root **`pnpm typegen`** regenerates TypeScript from schemas for consumers (`@bulqit/sanity-cms`).

**Features controlled:** Content modeling, document editing, and media used by public `[...slug]` pages and marketing.
