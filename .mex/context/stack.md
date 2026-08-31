---
name: stack
description: Technology stack, library choices, and the reasoning behind them. Load when working with specific technologies or making decisions about libraries and tools.
triggers:
  - "library"
  - "package"
  - "dependency"
  - "which tool"
  - "technology"
edges:
  - target: context/architecture.md
    condition: when a technology's role in the end-to-end flow matters
  - target: context/decisions.md
    condition: when the reasoning behind a tech choice is needed
  - target: context/conventions.md
    condition: when understanding how to use a technology in this codebase
  - target: context/session-indexing.md
    condition: when using Node SQLite or filesystem watching
grounds_to: []
last_updated: 2026-08-31
---

# Stack

## Core Technologies

- **Node.js 22+** — runtime; required by project documentation and by the built-in `node:sqlite` API used for indexing.
- **TypeScript 5.7** — primary implementation language, compiled as a project-reference workspace with `tsc --build`.
- **pnpm 10.30.3** — workspace package manager and command runner; root scripts coordinate package builds.
- **SQLite** — local durable session index, accessed through Node's synchronous built-in database API.

Claude Code input and derived forks use newline-delimited JSON (`.jsonl`).

## Key Libraries

- **vitest** 2.1 — test runner; tests use `describe`/`it`/`expect`, snapshots, synthetic sessions, and transcript fixtures.
- **eslint** 9 with `typescript-eslint` 8 — repository linting through the flat `eslint.config.js` entry point.
- **prettier** 3.4 — repository formatter; use the root `pnpm format` script.

## What We Deliberately Do NOT Use

- No web framework or service runtime — Glassbox is a local command-line inspector, not a network application.
- No ORM for the session index — `@glassbox/store` owns a small explicit SQLite schema and prepared statements.
- No in-place transcript editor — raw JSONL transformation is isolated in the Claude adapter and produces derived text for a new file.

## Version Constraints

- Node.js must be `>=22`; older versions do not provide the SQLite runtime used by `@glassbox/store`.
- TypeScript is `^5.7.2`, Vitest `^2.1.8`, ESLint `^9.17.0`, and Prettier `^3.4.2` in the root manifest.
- Corepack resolves pnpm `10.30.3` for this workspace.
