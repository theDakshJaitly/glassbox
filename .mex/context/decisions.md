---
name: decisions
description: Key architectural and technical decisions with reasoning. Load when making design choices or understanding why something is built a certain way.
triggers:
  - "why do we"
  - "why is it"
  - "decision"
  - "alternative"
  - "we chose"
edges:
  - target: context/architecture.md
    condition: when a decision relates to system structure
  - target: context/stack.md
    condition: when a decision relates to technology choice
  - target: context/transcript-analysis.md
    condition: when deciding where normalization or classification belongs
  - target: context/cleaning.md
    condition: when changing losslessness, cleaning tiers, or fork safety
grounds_to: []
last_updated: 2026-08-31
---

# Decisions

## Decision Log

### Normalize tool transcripts behind adapters

**Date:** 2026-06-05
**Status:** Active
**Decision:** Claude Code JSONL is normalized into the tool-neutral `@glassbox/core` session model before analysis or storage.
**Reasoning:** Transcript formats are tool-specific, while context, cost, reclaimable analysis, and indexing need stable contracts that future adapters can share.
**Alternatives considered:** Let each analyzer parse raw JSONL (rejected because it duplicates schema knowledge and prevents tool-neutral reuse).
**Consequences:** Raw Claude event fields belong only in `@glassbox/adapter-claude-code`; new tools require adapters, not forks of the analysis layer.

### Keep reconstruction factual and classification stateful

**Date:** 2026-06-05
**Status:** Active
**Decision:** Context reconstruction emits `unknown` segments from session facts; reclaimable analysis separately consults `RepoState` and assigns verdicts.
**Reasoning:** The same snapshot must support the x-ray and hygiene analysis, while only classification should depend on current filesystem state.
**Alternatives considered:** Classify while reconstructing (rejected because it couples a reusable value projection to I/O and time-sensitive verdicts).
**Consequences:** Add new segment sources in context reconstruction, add evidence rules in reclaimable analysis, and keep both independently testable.

### Clean by forking and tombstoning

**Date:** 2026-06-07
**Status:** Active
**Decision:** Lossless cleaning creates a new session and replaces only provably dead content with tombstones while retaining message and tool-pair structure.
**Reasoning:** Deleting blocks can orphan `tool_use`/`tool_result` pairs, and rewriting the source would risk user data; short tombstones preserve resumability and explain the gap.
**Alternatives considered:** In-place mutation (rejected as unsafe), deleting whole messages/results (rejected as structurally invalid), summarization as the default (rejected because it can lose needed information).
**Consequences:** The adapter owns raw rewrites; the CLI must validate before writing, confirm unless explicitly bypassed, use a fresh session id, and re-parse the result.

### Use an incremental local SQLite index

**Date:** 2026-06-05
**Status:** Active
**Decision:** Parsed sessions and `(mtime, size)` freshness fingerprints are cached in a local SQLite database.
**Reasoning:** Session listing should not repeatedly parse hundreds of append-only transcript files, and the data should remain local and operationally simple.
**Alternatives considered:** Re-scan/re-parse on every list (rejected as unnecessarily expensive), hosted storage (rejected because it breaks the local-first boundary).
**Consequences:** Sync skips unchanged files, project-scoped pruning cannot remove other projects, watcher events are debounced, and schema changes belong in `@glassbox/store`.

