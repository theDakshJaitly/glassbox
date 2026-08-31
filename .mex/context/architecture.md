---
name: architecture
description: How the major pieces of Glassbox connect and flow. Load when working on system design, integrations, or understanding how components interact.
triggers:
  - "architecture"
  - "system design"
  - "how does X connect to Y"
  - "integration"
  - "flow"
edges:
  - target: context/transcript-analysis.md
    condition: when changing transcript parsing, context reconstruction, cost, or reclaimable classification
  - target: context/cleaning.md
    condition: when changing clean/compact behavior or transcript fork safety
  - target: context/session-indexing.md
    condition: when changing the SQLite index, incremental sync, session listing, or file watching
  - target: context/stack.md
    condition: when specific technology details are needed
  - target: context/decisions.md
    condition: when understanding why the architecture is structured this way
grounds_to: []
last_updated: 2026-08-31
mex:
  id: mx_01M1C0Q69HRCD5JT845SBZPTAK
  type: architecture
  status: promoted
  revision: 1
  title: architecture
---

# Architecture

<!-- mex:entity
id: mx_01M1C0Q68HFXC11Q523AVG5KBD
type: component
status: promoted
revision: 1
-->
## System Overview

Claude Code JSONL on disk → `@glassbox/adapter-claude-code` discovery and normalization → `@glassbox/core` session model.
Normalized session → `@glassbox/analysis` cost accounting and resident-context reconstruction → filesystem-backed reclaimable classification.
Reports → `@glassbox/cli` renderers for `inspect`, `xray`, and `cost`.
Reclaimable report + snapshot → pure eviction/compaction plans → adapter-owned JSONL transformations.
Candidate fork → structural comparison with the original → confirmation → a new session-id-named JSONL file; the source remains untouched.
Separately, adapter discovery → `@glassbox/store` incremental sync → local SQLite metadata/model cache → `sessions` and `watch` workflows.

<!-- mex:entity
id: mx_01M1C0Q67JG225JWJNJW9Y6X82
type: component
status: promoted
revision: 1
-->
## Key Components

- **`@glassbox/core`** — tool-neutral domain types and ports for sessions, adapters, token counting, repository state, context snapshots, and costs; depends on no raw Claude format knowledge.
- **`@glassbox/adapter-claude-code`** — discovers and parses Claude Code JSONL, owns raw-format forking and validation, and converts transcript events into the core model.
- **`@glassbox/analysis`** — reconstructs attributed context, computes provider-actual costs, classifies reclaimable segments, and produces pure cleaning/compaction plans.
- **`@glassbox/store`** — persists normalized sessions and freshness fingerprints in SQLite and performs incremental sync/watch through the core `Adapter` port.
- **`@glassbox/cli`** — command dispatcher and terminal rendering; composes adapters, analysis, filesystem state, storage, validation, confirmations, and writes.

<!-- mex:entity
id: mx_01M1C0Q66F6RNBBGJBNGT37P3N
type: component
status: promoted
revision: 1
-->
## External Dependencies

- **Claude Code transcript storage** — current input source under `~/.claude/projects`; Glassbox discovers and reads `.jsonl` sessions locally.
- **Local filesystem state** — file existence and modification times are required to prove `gone` and `stale-drift` classifications.
- **SQLite** — local session index, opened through Node's built-in `node:sqlite`; default CLI path is `~/.glassbox/index.db`.
- **Anthropic API** — conditionally used by `bench` and Tier 3 compact summarization; requires `ANTHROPIC_API_KEY` and is not needed for inspection or lossless cleaning.

<!-- mex:entity
id: mx_01M1C0Q643NP1MCFCDW7A3NAYC
type: component
status: promoted
revision: 1
-->
## What Does NOT Exist Here

- No server, hosted database, browser UI, or telemetry path is part of the inspected application flow; the product is a local CLI.
- No adapter for Codex or other agent tools is implemented yet; the model and indexer are tool-neutral, but Claude Code is the only concrete adapter.
- No operation rewrites the user's original transcript; mutations target newly minted fork files only.

