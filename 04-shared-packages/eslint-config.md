# `packages/eslint-config`

## Paths

| Item | Path |
|------|------|
| Presets | `packages/eslint-config/node.mjs`, `next.mjs` |
| `package.json` **exports** | `"./node"`, `"./next"` |

## Behavior synopsis

Central **ESLint flat config** for Node packages vs Next/React apps: extends **`eslint-config-next`**, **TypeScript ESLint**, **import sorting**, **React hooks**, **React Compiler** plugin, **Prettier compatibility**.

**Features controlled:** Enforces consistent **lint rules** repo-wide when each package’s `eslint.config.*` extends `@bulqit/eslint-config/next` or `/node`.
