---
name: setup
description: Dev environment setup and commands. Load when setting up the project for the first time or when environment issues arise.
triggers:
  - "setup"
  - "install"
  - "environment"
  - "getting started"
  - "how do I run"
  - "local development"
edges:
  - target: context/stack.md
    condition: when specific runtime or tool versions are needed
  - target: context/architecture.md
    condition: when understanding what build and CLI commands compose
  - target: context/cleaning.md
    condition: when configuring API-backed compact or bench workflows
grounds_to: []
last_updated: 2026-08-31
mex:
  id: mx_01M1C0Q6RWXNKMPNYTXHSBYBQT
  type: guide
  status: promoted
  revision: 4
  title: setup
  relations:
    - type: related_to
      target: mx_01M1C0Q69HRCD5JT845SBZPTAK
      note: when understanding what build and CLI commands compose
    - type: related_to
      target: mx_01M1EXP2J6VE5VACV0N83WSG3J
      note: when specific runtime or tool versions are needed
    - type: related_to
      target: mx_01M1EXP1WA69DT2C57EJVHN8WT
      note: when configuring API-backed compact or bench workflows
---

# Setup

## Prerequisites

- Node.js `>=22`.
- pnpm `10.30.3` (resolved by Corepack for this workspace).
- Claude Code transcript files under `~/.claude/projects` for real inspection/indexing workflows.

## First-time Setup

1. `pnpm install`
2. `pnpm build`
3. `pnpm test`

<!-- mex:entity
id: mx_01M1C0Q6QSN6K10YW957DCAQJC
type: guide
status: promoted
revision: 1
-->
## Environment Variables

- Required: none for build, test, inspection, cost analysis, indexing, or lossless cleaning.
- `ANTHROPIC_API_KEY` (conditional) — required by `glassbox bench`; enables Tier 3 summarization during `glassbox compact`, which otherwise continues with Tiers 0–2.
- Optional: none currently documented.

<!-- mex:entity
id: mx_01M1C0Q6PM4Q6EM8PHT2119G0H
type: guide
status: promoted
revision: 1
-->
## Common Commands

- `pnpm lint` — run ESLint across the workspace.
- `pnpm typecheck` — run TypeScript project-reference checks with `tsc --build`.
- `pnpm test` — run the full Vitest suite once.
- `pnpm test:watch` — run Vitest in watch mode.
- `pnpm build` — type-build the workspace, then build the `glassbox` package.
- `pnpm clean` — clean TypeScript build output.
- `pnpm format` — rewrite repository formatting with Prettier.

<!-- mex:entity
id: mx_01M1C0Q6NFHNWV3FWWZ9TC7G76
type: guide
status: promoted
revision: 1
-->
## Common Issues

**No sessions found:** Real CLI workflows discover Claude Code sessions under `~/.claude/projects`; provide an explicit `.jsonl` locator or ensure Claude Code has created sessions there.

**`ANTHROPIC_API_KEY not set`:** Set the variable only when running `bench`. `compact` can still write its Tier 0–2 result without the key; Tier 3 is skipped.

**SQLite/index command fails on an older runtime:** Verify `node --version` is at least 22 before debugging the store; it uses Node's built-in `node:sqlite` API.
