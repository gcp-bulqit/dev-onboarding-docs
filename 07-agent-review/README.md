# Cursor / agent context (this folder)

This directory groups **AI assistant configuration** for this repository.

> **Git:** The entire `.cursor/` path is listed in `.gitignore` (private local/editor state). A **tracked mirror** for review lives under `external-review/07-agent-review/.cursor/`.

## Contents

| Path | Description |
|------|-------------|
| **`rules`** | Cursor **project rules**: Next.js/TypeScript/Tailwind/shadcn conventions, RSC preference, URL constants, methodology. |
| **`AGENTS.md`** | Copy of the repo root **`AGENTS.md`** — commands, architecture, migrations, testing safety, PR/commit rules, GitNexus skill index. |
| **`CLAUDE.md`** | Copy of the repo root **`CLAUDE.md`** — points agents at **`AGENTS.md`** in this folder (`@AGENTS.md`). |
| **`skills/gitnexus/*/SKILL.md`** | Copy of **`.claude/skills/gitnexus/`** (Claude Code GitNexus skills): CLI, guide, exploring, impact analysis, debugging, refactoring. |

## Canonical sources

- The **authoritative** `AGENTS.md` and `CLAUDE.md` remain at the **repository root** (for tools that expect them there).
- The **authoritative** GitNexus skills remain under **`.claude/skills/gitnexus/`** (Claude Code). After editing a skill, refresh copies here:

  ```bash
  # From repo root (PowerShell)
  Copy-Item -Path .claude\skills -Destination .cursor\skills -Recurse -Force
  ```

## Refresh the review bundle

After updating `.cursor/` or root agent files, sync the committed snapshot:

```bash
Copy-Item -Path .cursor\* -Destination external-review\07-agent-review\.cursor\ -Recurse -Force
```
