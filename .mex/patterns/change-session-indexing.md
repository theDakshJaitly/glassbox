---
name: change-session-indexing
description: Change the SQLite session schema, incremental sync, pruning, metadata listing, or live watch behavior.
triggers:
  - "session index"
  - "SQLite schema"
  - "index sync"
  - "watch sessions"
edges:
  - target: context/session-indexing.md
    condition: always load before changing store or indexing behavior
  - target: patterns/add-transcript-parsing.md
    condition: when normalized Session shape or adapter discovery changes
last_updated: 2026-08-31
mex:
  id: mx_01M1C0Q6XE1GZNSPFXJS56DQDJ
  type: pattern
  status: promoted
  revision: 2
  title: change-session-indexing
  grounds_to:
    - node: method:7ab7b4ee54d6774ba2738b45cbf95729
      fingerprint: mh:64:7b226d696e68617368223a5b373433353431352c39313731393030322c33313437323634352c33353634353436352c393330313935362c3130363239353232362c323337343236312c363837333631332c36333737373831322c38373939373330372c3139333230343036372c34383730353332392c34303334303630342c34323033363635302c32353330363039302c35393637383033382c31363837313936352c32353233373034342c3134383934313933392c32353534313330372c34333530373333342c32363737353237332c31313832313139322c35313533353937352c35303833383437372c32363630383036312c34323337393231382c31313730303339382c3239313133352c35333937323336322c31333539343332332c32343034333234312c31373536313039372c31323739333134302c34313937383934302c31383438363434382c33313335333138302c33313334383735312c32343433363432322c31383233333239312c34343739373538342c31353539383738382c32373334323233302c32313037393237362c32343839393338362c34373337313639322c31383833373332382c31373834373138312c3131313739383932362c3131333132303036392c393934393334312c33393833373439372c33333333333239362c31383234373139322c34393738303838392c34363630323735382c35323732302c31323139363435312c3130393933373435312c32323931323839392c31393036303032382c31373035323937342c353335333431392c31363630313236315d2c226e65696768626f7273223a5b2266756e6374696f6e3a3165303163656166363032356234356232343163373461376431393863613763222c2266756e6374696f6e3a3464393464313362656132666262653066353664336133373463653566333937222c2266756e6374696f6e3a3564623465336537363031316338336564303032666633663664653135663739222c2266756e6374696f6e3a6235376364316634343564326133303135306561636531396130623930313766222c226d6574686f643a3231393335613261613236356163666266633464386332623232313834613232222c226d6574686f643a3761323132376439396662633463323533656466623464313935653635393538222c226d6574686f643a6131663964373136343833646266353938626462363364316437393733656538222c226d6574686f643a6638313635306132346131343232366165333465393439653565303937656334225d2c22746f6b656e436f756e74223a3139377d
  relations:
    - type: related_to
      target: mx_01M1C0Q6SYGZ3YF2SMD1X89DAP
      note: when normalized Session shape or adapter discovery changes
---

# Change Session Indexing

## Context

The store caches normalized sessions and metadata locally. [`SessionIndexer.sync()`](mex://method:7ab7b4ee54d6774ba2738b45cbf95729) is the reconciliation contract: unchanged `(mtime, size)` entries are not parsed, failures are surfaced, and deletion pruning respects discovery scope.

## Steps

1. Determine whether the change belongs to `packages/store/src/session-index.ts` (schema/query/storage) or `packages/store/src/indexer.ts` (discovery reconciliation/watch).
2. Run `mex impact` on `SessionIndex` or `SessionIndexer::sync` before changing metadata or freshness semantics.
3. Keep metadata columns, row mapping, serialized model, and public types synchronized for schema changes; define compatibility/migration behavior explicitly.
4. Preserve the cheap fingerprint query and unchanged fast path.
5. Keep project filters consistent between discovery, listing, and pruning.
6. For watch changes, retain `.jsonl` filtering, per-locator debounce, initial sync, and explicit cleanup.
7. Extend `session-index.test.ts` or `indexer.test.ts` with in-memory SQLite/fake adapter cases; use a temporary directory only for real watch behavior.

## Gotchas

- A project-scoped discovery omits other projects by design; pruning all unseen locators would delete valid cache entries.
- Metadata lists must not parse/load every stored `model_json` blob.
- A parse failure should not masquerade as an unchanged or removed session.
- Watchers receive bursts and deletions; stat failure means removal only if the index actually contained that locator.
- Node `>=22` is required for `node:sqlite`.

## Verify

- [ ] First sync parses all; second unchanged sync parses none; one fingerprint change parses one.
- [ ] Missing sources prune only within the active scope.
- [ ] List ordering/filter/limit and row mapping remain correct.
- [ ] Watch initial/add/delete and cleanup tests pass.
- [ ] `pnpm test -- packages/store`, `pnpm typecheck`, and `pnpm lint` pass.

## Debug

Compare adapter discovery refs, stored fingerprints, and `SyncResult` counters. If watch differs from one-shot sync, inspect the constructed locator, debounce timer, and stat result before changing SQLite behavior.

## Update Scaffold

- [ ] Update `.mex/ROUTER.md` "Current Project State" if what's working/not built has changed
- [ ] Update any `.mex/context/` files that are now out of date
- [ ] If this is a new task type without a pattern, create one in `.mex/patterns/` and add to `INDEX.md`

