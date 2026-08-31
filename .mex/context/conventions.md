---
name: conventions
description: How code is written in Glassbox — naming, structure, patterns, and style. Load when writing new code or reviewing existing code.
triggers:
  - "convention"
  - "pattern"
  - "naming"
  - "style"
  - "how should I"
  - "what's the right way"
edges:
  - target: context/architecture.md
    condition: when a convention depends on understanding package responsibilities
  - target: context/transcript-analysis.md
    condition: when adding parser or analyzer behavior and tests
  - target: context/cleaning.md
    condition: when a change can affect transcript safety or losslessness
  - target: patterns/add-transcript-parsing.md
    condition: when extending Claude Code normalization
  - target: patterns/change-context-analysis.md
    condition: when adding a context source, verdict, or cost calculation
grounds_to: []
last_updated: 2026-08-31
---

# Conventions

## Naming

- Files and package folders use kebab-case (`context-snapshot.ts`, `session-index.ts`, `adapter-claude-code`).
- Functions and methods use camelCase with action-oriented names (`parseClaudeSession`, `reconstructContext`, `planEviction`, `pruneMissing`); classes and interfaces use PascalCase.
- Tests are co-located with implementation as `*.test.ts`; durable parser fixtures live under the package's `test/fixtures/` directory.
- Core identifiers use branded types and `as…` constructors (`asSessionId`, `asMessageId`, `asIsoTimestamp`) instead of passing arbitrary strings across model boundaries.
- Constants that define policy sets use upper snake case (`PROVABLE_CLASSES`, `TIER1_CLASSES`, `RECLAIMABLE_STATUSES`).

## Structure

- `packages/core` owns tool-neutral model contracts and ports; it must not learn Claude Code JSONL details or touch filesystem/git to classify data.
- `packages/adapter-claude-code` owns discovery, raw-event normalization, JSONL rewriting, and transcript validation.
- `packages/analysis` owns context/cost/reclaimable calculations and pure planning; external state enters through narrow ports such as `RepoState` and `TokenCounter`.
- `packages/store` owns SQLite persistence and adapter-driven indexing; CLI code should compose it rather than duplicating queries or freshness logic.
- `packages/cli` is the orchestration and rendering boundary. Keep reusable parsing, analysis, transformation, and storage logic in their owning packages.
- ESM TypeScript imports use emitted `.js` specifiers, including relative imports such as `./parse.js`.

## Patterns

Keep raw-format concerns behind the adapter boundary:

```ts
// Correct: normalize once, then analyze the core Session.
const session = await adapter.parse(ref);
const result = await analyzeSessionReclaimable(session, options);

// Wrong: make analysis walk Claude Code JSONL fields directly.
```

Separate facts, verdicts, planning, and I/O:

```ts
const snapshot = reconstructContext(session, { tokens });
const { report } = await analyzeSessionReclaimable(session, { repo, tokens });
const plan = planEviction(report, snapshot);
const fork = forkTranscript(raw, evictions, { newSessionId });
```

Provider usage is authoritative for cost; the injected `TokenCounter` estimate is only for content that the transcript does not size. Never silently present estimated token math as provider actuals.

Transcript transformations must preserve JSONL lines they do not understand, maintain `tool_use`/`tool_result` pairs and `parentUuid` structure, compare validation problems before/after, and write only a new session file.

## Verify Checklist

Before presenting any code:

- [ ] Package ownership is preserved: core contracts, Claude raw format, analysis, store, and CLI orchestration remain separated.
- [ ] New external inputs are typed or validated at their boundary; no unexplained `any` was introduced.
- [ ] Parser/analysis/index changes include co-located Vitest coverage and fixtures or synthetic sessions for edge cases.
- [ ] Transcript transformations preserve pairing/threading, do not mutate the original, and are checked with `validateTranscript` plus a re-parse.
- [ ] `pnpm lint` passes.
- [ ] `pnpm typecheck` passes.
- [ ] `pnpm test` passes; run `pnpm build` for package/export or CLI changes.
- [ ] `DOC.md` and `CHANGELOG.md` reflect every user-visible or developer-visible change.

