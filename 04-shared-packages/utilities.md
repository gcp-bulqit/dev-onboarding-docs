# `packages/utilities`

## Paths

| Export subpath | Role |
|----------------|------|
| `@bulqit/utilities/strings` | String helpers |
| `@bulqit/utilities/numbers` | Numeric parsing/formatting |
| `@bulqit/utilities/environment` | Env detection helpers |
| `@bulqit/utilities/booleans` | Boolean coercion / checks |

## Behavior synopsis

Small **dependency-free** (or nearly) utilities imported everywhere to avoid duplicating parsing logic between server components, workers, and tests.

**Features controlled:** Not user-facing features directly—**code quality and consistency** across the monorepo.
