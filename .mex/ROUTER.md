---
name: router
description: Session bootstrap and navigation hub. Read at the start of every session before any task. Contains project state, routing table, and behavioural contract.
edges:
  - target: context/architecture.md
    condition: when working on system design, integrations, or understanding how components connect
  - target: context/stack.md
    condition: when working with specific technologies, libraries, or making tech decisions
  - target: context/conventions.md
    condition: when writing new code, reviewing code, or unsure about project patterns
  - target: context/decisions.md
    condition: when making architectural choices or understanding why something is built a certain way
  - target: context/setup.md
    condition: when setting up the dev environment or running the project for the first time
  - target: context/transcript-analysis.md
    condition: when working on Claude Code parsing, context reconstruction, costs, or reclaimable classification
  - target: context/cleaning.md
    condition: when working on clean/compact, forks, tombstones, or integrity validation
  - target: context/session-indexing.md
    condition: when working on SQLite storage, incremental sync, indexed session listing, or watching
  - target: patterns/INDEX.md
    condition: when starting a task — check the pattern index for a matching pattern file
last_updated: 2026-08-31
---

# Session Bootstrap

If you haven't already read `AGENTS.md`, read it now — it contains the project identity, non-negotiables, and commands.

Then read this file fully before doing anything else in this session.

## Current Project State

**Working:**

- Claude Code session discovery and JSONL normalization into a tool-neutral session model.
- CLI inspection flows for context composition, provider-actual cost, reclaimable context, and combined dashboards.
- Provable lossless cleaning into new resumable session forks, with structural validation and post-write re-parsing.
- Tiered compaction, including spent-output clearing, cold bulk trimming, optional API summarization, and comparison benchmarks.
- Incremental local SQLite indexing, metadata listing, project scoping, and debounced live watching.

**Not yet built:**

- Concrete transcript adapters for tools other than Claude Code.
- A provider-accurate tokenizer to replace the directional four-characters-per-token estimate for attributed content.
- Branch/subagent-specific context reconstruction; sidechains are currently preserved but analyzed linearly.

**Known issues:**

- Attributed context totals exclude the system prompt and tool definitions because Claude transcripts do not contain them.
- Unknown model identifiers remain unpriced, so some cost and wasted-per-turn values can be `null`.
- The local pricing table is dated and needs manual refresh when provider rates or model ids change.

## Routing Table

Load the relevant file based on the current task. Always load `context/architecture.md` first if not already in context this session.

| Task type | Load |
|-----------|------|
| Understanding how the system works | `context/architecture.md` |
| Working with a specific technology | `context/stack.md` |
| Writing or reviewing code | `context/conventions.md` |
| Making a design decision | `context/decisions.md` |
| Setting up or running the project | `context/setup.md` |
| Parsing transcripts or changing context/cost/reclaimable analysis | `context/transcript-analysis.md` |
| Changing cleaning, compact tiers, fork writing, or integrity checks | `context/cleaning.md` |
| Changing SQLite storage, indexing, session listing, or watching | `context/session-indexing.md` |
| Any specific task | Check `patterns/INDEX.md` for a matching pattern |

## Behavioural Contract

For every task, follow this loop:

1. **CONTEXT** — Load the relevant context file(s) from the routing table above. Check `patterns/INDEX.md` for a matching pattern. If one exists, follow it. Narrate what you load: "Loading architecture context..."
2. **BUILD** — Do the work. If a pattern exists, follow its Steps. If you are about to deviate from an established pattern, say so before writing any code — state the deviation and why.
3. **VERIFY** — Load `context/conventions.md` and run the Verify Checklist item by item. State each item and whether the output passes. Do not summarise — enumerate explicitly.
4. **DEBUG** — If verification fails or something breaks, check `patterns/INDEX.md` for a debug pattern. Follow it. Fix the issue and re-run VERIFY.
5. **GROW** — After meaningful work, run this binary checklist:
   - **Ground:** What changed in reality? Name the changed behavior, system, command, dependency, or workflow.
   - **Record:** If project state changed, update the "Current Project State" section above. If documented facts changed, update the relevant `context/` file surgically.
   - **Orient:** If this task can recur and no pattern exists, create one in `patterns/` using `patterns/README.md`, then add it to `patterns/INDEX.md`. If a pattern exists but you learned a gotcha, update it.
   - **Write:** Bump `last_updated` in every scaffold file you changed. If the why matters, run `mex log --type decision "<what changed and why>"` or `mex log "<note>"`.
