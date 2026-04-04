# Vercel (web app)

## Paths

| Item | Path |
|------|------|
| App | `apps/web/` — package name **`bulqit-website`** in `apps/web/package.json` |
| Build orchestration | Root `package.json` + `turbo.json` |
| CI/deploy docs | Root `README.md` |

## Behavior synopsis

- **Vercel** runs **`turbo run build --filter=bulqit-website...`** (or equivalent) on push; **Turbo remote cache** speeds rebuilds.
- **Environment variables** from the Vercel project are injected into serverless/edge functions and prefixed **`NEXT_PUBLIC_*`** values reach the browser.
- **Database migrations** run as a Turbo build dependency: in `turbo.json`, the `bulqit-website#build` task depends on `@bulqit/database#migrate`, so migrations run before the web app build. Note: the root `README.md` in the code repo still references a `postinstall` migration hook; that is outdated.

**Features controlled:** Hosting for **all user-facing HTTP** except long-lived worker processes.
