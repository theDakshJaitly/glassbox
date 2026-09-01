---
name: session-indexing
description: Local SQLite session storage, incremental reconciliation, and live transcript watching. Load for index/sessions/watch work.
triggers:
  - "index"
  - "SQLite"
  - "sessions"
  - "watch"
  - "incremental sync"
edges:
  - target: context/architecture.md
    condition: when the adapter, store, and CLI composition needs context
  - target: context/stack.md
    condition: when Node or SQLite runtime constraints matter
  - target: context/transcript-analysis.md
    condition: when normalized Session fields or adapter behavior change
  - target: patterns/change-session-indexing.md
    condition: when changing the schema, sync rules, pruning, or watch behavior
last_updated: 2026-08-31
mex:
  id: mx_01M1EXP2ABM1P7M325JHB1CHE7
  type: architecture
  status: promoted
  revision: 5
  title: session-indexing
  grounds_to:
    - node: method:7ab7b4ee54d6774ba2738b45cbf95729
      fingerprint: mh:64:7b226d696e68617368223a5b373433353431352c39313731393030322c33313437323634352c33353634353436352c393330313935362c3130363239353232362c323337343236312c363837333631332c36333737373831322c38373939373330372c3139333230343036372c34383730353332392c34303334303630342c34323033363635302c32353330363039302c35393637383033382c31363837313936352c32353233373034342c3134383934313933392c32353534313330372c34333530373333342c32363737353237332c31313832313139322c35313533353937352c35303833383437372c32363630383036312c34323337393231382c31313730303339382c3239313133352c35333937323336322c31333539343332332c32343034333234312c31373536313039372c31323739333134302c34313937383934302c31383438363434382c33313335333138302c33313334383735312c32343433363432322c31383233333239312c34343739373538342c31353539383738382c32373334323233302c32313037393237362c32343839393338362c34373337313639322c31383833373332382c31373834373138312c3131313739383932362c3131333132303036392c393934393334312c33393833373439372c33333333333239362c31383234373139322c34393738303838392c34363630323735382c35323732302c31323139363435312c3130393933373435312c32323931323839392c31393036303032382c31373035323937342c353335333431392c31363630313236315d2c226e65696768626f7273223a5b2266756e6374696f6e3a3165303163656166363032356234356232343163373461376431393863613763222c2266756e6374696f6e3a3464393464313362656132666262653066353664336133373463653566333937222c2266756e6374696f6e3a3564623465336537363031316338336564303032666633663664653135663739222c2266756e6374696f6e3a6235376364316634343564326133303135306561636531396130623930313766222c226d6574686f643a3231393335613261613236356163666266633464386332623232313834613232222c226d6574686f643a3761323132376439396662633463323533656466623464313935653635393538222c226d6574686f643a6131663964373136343833646266353938626462363364316437393733656538222c226d6574686f643a6638313635306132346131343232366165333465393439653565303937656334225d2c22746f6b656e436f756e74223a3139377d
  relations:
    - type: related_to
      target: mx_01M1C0Q69HRCD5JT845SBZPTAK
      note: when the adapter, store, and CLI composition needs context
    - type: related_to
      target: mx_01M1EXP2J6VE5VACV0N83WSG3J
      note: when Node or SQLite runtime constraints matter
    - type: related_to
      target: mx_01M1EXP2TK06FQH01Y1S5691Z8
      note: when normalized Session fields or adapter behavior change
    - type: related_to
      target: mx_01M1C0Q6XE1GZNSPFXJS56DQDJ
      note: when changing the schema, sync rules, pruning, or watch behavior
---

# Session Indexing

## Storage Model

`SessionIndex` opens a local SQLite database, creates the session schema, stores both metadata and the serialized normalized `Session`, and exposes metadata-only listing ordered newest-ended first. The default CLI path is `~/.glassbox/index.db`; `index`, `sessions`, and `watch` accept `--db`.

Each record keeps source modification time and size as a freshness fingerprint. This index is a cache of local transcript-derived data, not the source of truth.

## Reconciliation

[`SessionIndexer.sync()`](mex://method:7ab7b4ee54d6774ba2738b45cbf95729) discovers through the tool-neutral `Adapter`, skips a locator when both `(mtime, size)` match, re-parses and upserts new/changed sessions, surfaces parse failures, and prunes missing sources only within the discovery scope.

`watch()` performs an initial sync, recursively watches configured roots, filters to `.jsonl`, debounces rapid events per locator (250 ms by default), reindexes existing files, and removes vanished files. Callers own watcher and database shutdown.

## Invariants

- Never load the serialized model blob for list/freshness operations when metadata columns suffice.
- The unchanged fast path must not call `Adapter.parse`.
- A project-scoped sync must never prune sessions belonging to another project.
- Parse failures are returned/emitted with locators; do not swallow them or delete the prior entry implicitly.
- Watch changes must remain debounced because Claude Code appends in bursts.
- Always close `SessionIndex` in `finally`; the long-running watch command closes both watcher and index on `SIGINT`.

## Tests

- Use `SessionIndex.open(":memory:")` for deterministic store/indexer unit tests.
- Fake adapters should count parses and control fingerprints to prove new/changed/unchanged behavior.
- Keep a project-scoped prune regression test.
- Live watcher coverage uses a temporary directory and waits for observable index state; cleanup closes the watcher/index and removes the directory.

