---
name: agents
description: Always-loaded project anchor. Read this first. Contains project identity, non-negotiables, commands, and pointer to ROUTER.md for full context.
last_updated: 2026-08-31
---

# Glassbox

## What This Is
Glassbox is a local-first CLI that inspects Claude Code context, reports reclaimable cost, and writes validated cleaned session forks without modifying originals.

## Non-Negotiables
- Never modify a user's original transcript; cleaning writes a fresh session-id-named fork.
- Keep Claude JSONL details in `@glassbox/adapter-claude-code`; core, analysis, and store remain tool-neutral.
- Preserve provider-actual usage and transcript structure, especially tool pairs and `parentUuid` lineage.
- Keep `DOC.md` and `CHANGELOG.md` current for developer-visible or user-visible changes.

## Commands
- Dev: `pnpm test:watch` (no long-running app dev server)
- Test: `pnpm test`
- Lint: `pnpm lint`
- Typecheck: `pnpm typecheck`
- Build: `pnpm build`

## Code Graph

Use `.mex/graph.db` for implementation discovery: `mex graph scope "<task>" --fingerprint`, `mex graph get <id> --detail source`, exact-symbol graph queries, and `mex impact`. Treat graph source as already read. If the graph is unavailable, do not invent ids, fingerprints, or behavioral grounding.

## Scaffold Growth
After meaningful work, run GROW:
- Ground: what changed in reality?
- Record: update `ROUTER.md` and relevant `context/` files
- Orient: create or update a `patterns/` runbook if this can recur
- Write: bump `last_updated` on changed scaffold files and run `mex log` when rationale matters

The scaffold grows from real work, not just setup. See the GROW step in `ROUTER.md` for details.

## Navigation
At the start of every session, read `ROUTER.md` before doing anything else.
For full project context, patterns, and task guidance — everything is there.
