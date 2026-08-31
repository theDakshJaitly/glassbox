# Glassbox Documentation

Glassbox is an observability tool for AI agent memory and context. It is intended to provide a local-first, cross-tool x-ray and hygiene monitor for coding agents.

## Purpose

Use this document as the complete app documentation page. Any meaningful change to behavior, commands, configuration, architecture, public APIs, or workflows must update this file.

## Quick start

```bash
pnpm install
pnpm build
pnpm test
```

## Common commands

```bash
pnpm lint        # Run lint checks
pnpm typecheck   # Run TypeScript project checks
pnpm test        # Run tests
pnpm build       # Build all packages
pnpm clean       # Clean TypeScript build output
```

## Project structure

- `packages/` — workspace packages.
- `docs/` — supporting documentation.
- `assets/` — project assets.
- `agent-docs/` — mandatory coding-agent guidelines.
- `AGENTS.md` — root instruction file binding agents to the guidelines.
- `.mex/` — AI context scaffold with a session router, project/domain context, grounded task runbooks, and a generated code graph.
- `.gitattributes` — enforces LF line endings for all tracked files under `.mex/`.
- `.github/pull_request_template.md` — pull request checklist for docs, changelog, validation, and review notes.

## AI context scaffold

Coding agents should begin with `.mex/ROUTER.md`, load the routed context and matching pattern, and use `.mex/graph.db` through `mex graph` for implementation discovery. The scaffold contains dedicated context for transcript analysis, cleaning/compaction safety, and local session indexing. Behavioral claims in deep-domain and pattern files are grounded to code-graph node ids and fingerprints so drift can be detected.

The generated `.mex/graph.db` and its SQLite/MEX sidecars are local artifacts ignored by Git; rebuild them with `mex graph` when needed.

When behavior or architecture changes, update the affected `.mex/context/` file, the router's current project state when applicable, and any recurring-task pattern. Keep scaffold `last_updated` fields and cross-file edges current.

## Documentation requirements

When adding or changing features, update this document with:

- What changed.
- How users or developers use it.
- Any new commands, options, configuration, or environment variables.
- Any migration notes or known limitations.

## Troubleshooting

If a command fails, first verify:

1. Node.js version is `>=22`.
2. Dependencies were installed with `pnpm`.
3. The workspace is clean and package scripts are run from the repository root.

## Known limitations

- This document is currently a baseline and should be expanded as app features are implemented or changed.
