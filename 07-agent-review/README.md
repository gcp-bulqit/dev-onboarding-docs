# Agent configuration (repo snapshot)

This directory was a snapshot of AI assistant configuration from the code repo. Parts of it are stale.

## What exists on staging

| Path | Status |
|------|--------|
| `AGENTS.md` (root) | Active. Commands, architecture overview, migration rules, testing safety, PR/commit conventions. |
| `CLAUDE.md` (root) | Active. Points agents to `AGENTS.md`. |
| `.cursor/rules` | Active. Cursor project rules (Next.js/TypeScript/Tailwind conventions). |
| `.claude/skills/gitnexus/` | All 6 skill files present (CLI, guide, exploring, impact analysis, debugging, refactoring). |

## What does not exist on staging

| Path | Status |
|------|--------|
| `.cursor/AGENTS.md`, `.cursor/CLAUDE.md`, `.cursor/skills/` | Not tracked. Only `.cursor/rules` exists. |
| `external-review/` | Not tracked. |
| `.context/history/` | Not tracked. Session checkpoint workflow in AGENTS.md references this path. |
| `memory/batches.md` | Not tracked. ClickUp batch tracking in AGENTS.md references this path. |

## AGENTS.md content notes

The `AGENTS.md` in this directory is a snapshot that has diverged from the repo root version. It contains:

- **Useful for any developer:** Common commands, architecture overview, database migration workflow, testing conventions and safety warnings (integration tests truncate tables), PR guidelines.
- **FW-specific process:** ClickUp task fix workflow, session checkpoint workflow, batch ordering, commit convention (`CU-{taskID}`). These are FutureWorks internal process and will not apply after handoff.
- **GitNexus references:** The file includes a GitNexus MCP section referencing `.claude/skills/gitnexus/` skills. All six skill files exist on staging.
