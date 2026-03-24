# AGENTS.md

This file provides guidance to coding agents when working in this repository.

## Common Commands

### Development

```bash
pnpm dev                    # Start all apps in dev mode
pnpm dev --filter=bulqit-website  # Start only the web app
```

### Building

```bash
pnpm build                  # Build all packages
pnpm build --force          # Force rebuild (ignore cache)
pnpm build:packages         # Build only shared packages
```

### Testing

```bash
# From apps/web/
pnpm test                   # Run all tests
pnpm test:unit              # Unit tests only
pnpm test:integration       # Integration tests only
pnpm vitest run src/path/to/file.test.tsx  # Single test file
pnpm vitest --watch         # Watch mode
```

### Code Quality

```bash
pnpm lint                   # ESLint across all packages
pnpm format                 # Prettier formatting
pnpm check-types            # TypeScript type checking
```

### Database (from packages/database/)

```bash
pnpm drizzle generate       # Generate migration from schema changes
pnpm migrate                # Run migrations
```

### Type Generation

```bash
pnpm typegen                # Generate Sanity CMS types
```

## Architecture Overview

### Monorepo Structure

- **apps/web** - Next.js 16 website (main application)
- **apps/temporal-workers** - Temporal workflow engine workers
- **apps/worker-notifications** - Notification worker
- **apps/worker-zoho-sync** - Zoho CRM sync worker
- **packages/database** - Drizzle ORM schemas and migrations (PostgreSQL/Neon)
- **packages/utilities** - Shared helper functions
- **packages/messaging** - Messaging service abstractions
- **packages/notifications** - Notification system
- **packages/temporal-workflows** - Temporal workflow definitions
- **packages/sanity-cms** - Sanity CMS schema and utilities

### Web App Structure (apps/web/src/)

- **app/** - Next.js App Router (pages, layouts, API routes)
- **actions/** - Server actions organized by feature (admin/, dashboard/, add-ons/)
- **components/** - React components organized by feature
- **hooks/** - Custom React hooks
- **lib/** - Utility functions
- **consts/** - Application constants (URLs, metadata)
- **types/** - TypeScript type definitions

### Key Patterns

- Server Actions over API routes for mutations
- React Server Components preferred; minimize `'use client'`
- Radix UI + shadcn/ui for components
- Redux Toolkit for global state
- Tailwind CSS using design tokens from config (no hardcoded values)

## Database Migrations (Drizzle)

**ALWAYS use `pnpm drizzle generate` - NEVER write migrations manually.**

**NEVER use `drizzle push` without EXPLICITLY asking first. It can drop data and is a DANGEROUS operation.**

1. Edit schema files in `packages/database/src/schemas/`
2. Export new schemas in `packages/database/src/schemas/index.ts`
3. Run `pnpm drizzle generate` from `packages/database/`
4. Rename the generated migration to something meaningful
5. **Verify migrations are in order in `migrations/meta/_journal.json`** - migrations must be properly chained

### Why This Matters

Drizzle maintains snapshots that track the full schema state. Manual migrations break snapshot chains, causing duplicate migrations on future generates. Migrations must always be properly chained in the journal.

### Verification

After migration changes, run `pnpm drizzle generate`. Expected output: `No schema changes, nothing to migrate`. If it generates a new migration, your snapshots are out of sync.

**Always verify the journal ordering** - migrations in `_journal.json` must be sequential and properly indexed.

### Merging Branches with Migrations

When staging has new migrations:

1. Merge staging into your branch
2. If conflicts in `migrations/meta/_journal.json`: accept staging's migrations, renumber yours after
3. Delete your old migration files and snapshots
4. Run `pnpm drizzle generate` to regenerate with correct snapshots
5. **Verify the journal is in proper order** - no gaps, no duplicates, sequential indices

## Testing Conventions

- Unit tests: `*.test.{ts,tsx}` files
- Integration tests: `*.integration.test.{ts,tsx}` files
- Use `datatest-id` attribute (not `data-testid`)
- Test files typically in `_tests/` subdirectories

### CRITICAL: Integration Test Safety

**NEVER run `pnpm test:integration` (from `apps/web`) without verifying the database first.**

Integration tests run `TRUNCATE TABLE ... CASCADE` on ALL tables. If `DB_URL` points to a shared database (staging/production), this will **wipe all data**.

Before running integration tests:
1. Check `DB_URL` in `apps/web/.env.local`
2. Ensure it points to a **personal dev branch**, NOT staging
3. Staging DB contains: `ep-raspy-leaf-a69ycsvz` - NEVER use this for tests

From `apps/web`:
- Safe: `pnpm test:unit` (no database interaction)
- Dangerous: `pnpm test:integration` (wipes the database)

## Pull Request Guidelines

**CRITICAL: Follow these rules when creating PRs.**

### Target Branch

- **ALWAYS target `development`** - never `main` or `staging`
- PRs to `main` require explicit user approval

### PR Description Format

- Write clear, concise descriptions summarizing the changes
- Include a "Test plan" section with verification steps
- **NEVER include "Generated with Claude Code" or similar AI attribution taglines**
- **NEVER include links to claude.ai or similar**

Example format:
```markdown
## Summary
- Brief description of changes

## Test plan
- [ ] Verification step 1
- [ ] Verification step 2
```

## Commits

**NEVER add "Co-Authored-By: Claude Code" or similar AI attribution to commit messages.**

- Write clear, conventional commit messages (e.g., `feat:`, `fix:`, `docs:`, `chore:`)
- Keep the subject line concise (50 chars or less)
- No AI attribution or references to Claude in commits

## URL Constants

Use constants from `apps/web/src/consts/urls.ts` for all internal URLs.

## Session Checkpoints

Maintain session checkpoints in `.context/history/` to persist progress across sessions.

### At Session Start

1. Create `.context/history/` if it doesn't exist
2. Read the 1-2 most recent session files for context on prior work
3. When starting actual work, create `session_YYYY-MM-DD-HH-MM.md` with current timestamp

### During Work

Update the session file after completing each task from a plan:

```markdown
# Session: YYYY-MM-DD HH:MM

## Goal

[Current objective]

## Completed

- [x] [HH:MM] Task description

## Files Modified

- `path/to/file.tsx` - Brief description

## Next Steps

- Remaining work or blockers
```

### Guidelines

- Use `date +%H:%M` to get current time for timestamps
- Don't create checkpoints for quick Q&A without actual work
- Update Files Modified as you edit files
- Update Next Steps before ending work session

## Bulqit ClickUp Task Fix Workflow

### Workflow Per Task

1. Fetch task details from ClickUp
2. **Read task comments** — always fetch comments alongside task details to get the full picture (QA failure notes, additional scope, clarifications from the team)
3. **Check task attachments** for screenshots or videos showing the bug
4. Update ClickUp status to **"in progress"** when starting work
5. Investigate the bug, propose a fix to the user
6. Implement only after user approval
7. Commit and push
8. **Never move to the next task** without user's explicit permission
9. If user says skip: do NOT update ClickUp status (leave as-is)
10. If status was already changed to "in progress" before skipping: revert to "release 3"

### Batch Ordering

Tasks are organized into batches tracked in `memory/batches.md`. Always follow batch order — complete all tasks in the current batch before moving to the next.

### Commit Convention

- Message format: `CU-{taskID}` (e.g., `CU-868hppv4m`)
- No other text, no Co-Authored-By, no AI attribution
- No need to confirm commit message with user

### Branch & PR Convention

- Branch off `development` (not `main`)
- Target PRs to `development` (not staging, not main)
- Branch naming: `fix/batch-{N}-{scope}` (e.g., `fix/batch-2-member-dashboard`)
- No AI attribution in PR descriptions

### Code Changes

- Apply changes in the active repository root (the current project), not an alternate worktree unless explicitly requested.
- Run git operations (add, commit, push) from this same repository root.

### Post-Merge

- When PR is merged: update completed tasks to "on staging" (only if actual dev work happened)

<!-- gitnexus:start -->
# GitNexus MCP

This project is indexed by GitNexus as **bul-website** (11897 symbols, 36445 relationships, 300 execution flows).

## Always Start Here

1. **Read `gitnexus://repo/{name}/context`** — codebase overview + check index freshness
2. **Match your task to a skill below** and **read that skill file**
3. **Follow the skill's workflow and checklist**

> If step 1 warns the index is stale, run `npx gitnexus analyze` in the terminal first.

## Skills

| Task                                         | Read this skill file                                        |
| -------------------------------------------- | ----------------------------------------------------------- |
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md`       |
| Blast radius / "What breaks if I change X?"  | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?"             | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md`       |
| Rename / extract / split / refactor          | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md`     |
| Tools, resources, schema reference           | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md`           |
| Index, status, clean, wiki CLI commands      | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md`             |

> **Cursor:** The same GitNexus skill files are copied to `.cursor/skills/gitnexus/` (see `external-review/07-agent-review/README.md`). Edit skills under `.claude/skills/` first, then refresh the copy.

<!-- gitnexus:end -->