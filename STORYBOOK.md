# Storybook

**Hosted (development branch):** [https://storybook.example.com/](https://storybook.example.com/)

## Local

From `apps/web`:

```bash
pnpm storybook
```

Port **6006** — see `apps/web/package.json` (`storybook`, `storybook:dev`, `storybook:css`).

## Configuration

- `apps/web/.storybook/main.ts` — Stories: `../src/**/*.stories.tsx`; Vite aliases for `@/` and workspace packages (`@bulqit/notifications`, `@bulqit/database`, `@bulqit/messaging`, etc.).
- `apps/web/.storybook/preview.tsx` — Global decorators.

## Linking to a story

1. Open the hosted Storybook and use the sidebar.
2. Copy the URL’s `path` query (e.g. `?path=/story/shared-buttons-primarybutton--default`).
3. Story `title` strings are defined in `apps/web/src/**/_stories/*.stories.tsx`.

Story id pattern: **slugified `title` + `--` + slugified story name** (often `Default` → `--default`).

## Deploy

`pnpm build-storybook` in `apps/web` produces static output; the hosted instance tracks the **development** branch on Vercel.
