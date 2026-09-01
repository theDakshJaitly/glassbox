---
name: add-transcript-parsing
description: Extend Claude Code JSONL normalization for a new raw event, block, tool, or metadata shape.
triggers:
  - "Claude transcript"
  - "parser"
  - "JSONL event"
  - "tool result"
edges:
  - target: context/transcript-analysis.md
    condition: always load before changing normalization behavior
  - target: patterns/debug-transcript-fork.md
    condition: when the parser change also affects fork validation or re-parsing
last_updated: 2026-08-31
mex:
  id: mx_01M1C0Q6SYGZ3YF2SMD1X89DAP
  type: pattern
  status: promoted
  revision: 3
  title: add-transcript-parsing
  grounds_to:
    - node: function:0efb013f713a5e702da7fe74750295d3
      fingerprint: mh:64:7b226d696e68617368223a5b313233343839342c35363934373039312c33313437323634352c31363535323639322c393330313935362c33323130333633352c323731343330352c32373331313232302c31343737303630362c31343439333930392c33393539393237332c33343532333931312c34303334303630342c31363539313035372c32353330363039302c35313239393334352c31363837313936352c323132363738322c32383238393138372c3234313939302c393932323138332c32363737353237332c31313832313139322c32373236393136392c33303130303035392c31393831303431322c33373336303731332c31313730303339382c3239313133352c35323135393631352c31333539343332332c37343239303535372c31373536313039372c363938343133322c34313937383934302c31333637383833372c33363031373336322c33313334383735312c323035383836302c33343839333632372c31363039323330362c31353539383738382c363733393930352c32313037393237362c31393537393830372c393530353039362c31383833373332382c33363431323237382c3131313739383932362c34333732303033302c393934393334312c33393833373439372c383338373836352c31383234373139322c34303630373134322c373936373930382c35323732302c393539393838362c31323432303936302c31353736323435352c31353831373536302c31373035323937342c33383337343331332c333934373937315d2c226e65696768626f7273223a5b2266756e6374696f6e3a3131366363656435303438323661656361303162393436356432383632393130222c2266756e6374696f6e3a3564343166633735643562346363336364636432373866636665326435346130222c2266756e6374696f6e3a3565353430393738643932343864663365333431393732346536616166633436222c2266756e6374696f6e3a3733393934626133306166383464343662633064333139666661663962316638222c2266756e6374696f6e3a3766643531333832623662616466366234616261646564313135373963323738222c2266756e6374696f6e3a3933323064313733396133373230666162653466613538623131653538363732222c2266756e6374696f6e3a6165346662366535643936636438633130343636663630353930323239343535222c2266756e6374696f6e3a6237303433306430343635303934663462313335323863653038343666363435222c2266756e6374696f6e3a6431613334656232356662353936653162366636666532356230623366333761222c2266756e6374696f6e3a6639333331393161323266386661653263616339623866643036316431663938222c226d6574686f643a3931393539663636343063336666613031383037393239663261323539353931225d2c22746f6b656e436f756e74223a3534397d
  relations:
    - type: related_to
      target: mx_01M1C0Q6YP6H9RQ4AG09NWK6MM
      note: when the parser change also affects fork validation or re-parsing
    - type: related_to
      target: mx_01M1EXP2TK06FQH01Y1S5691Z8
      note: always load before changing normalization behavior
---

# Add Transcript Parsing

## Context

Load `context/transcript-analysis.md`. Raw Claude Code schema knowledge belongs in `packages/adapter-claude-code`; downstream code consumes the core `Session` produced by [`parseClaudeSession()`](mex://function:0efb013f713a5e702da7fe74750295d3).

## Steps

1. Resolve the affected parser/helper symbol with `mex graph query where-defined` and inspect callers before editing.
2. Extend the raw event/block typing and normalization in `packages/adapter-claude-code/src/`; do not leak raw fields into `packages/core` unless they represent a tool-neutral concept.
3. Preserve tree `uuid`/`parentUuid`, role, timestamp, sidechain, model, provider message id, and warnings as applicable.
4. If the shape is a tool interaction, keep request/result stitching and lift file or memory operations through the existing normalized records.
5. Add the smallest anonymized case to `packages/adapter-claude-code/test/fixtures/basic-session.jsonl` or construct a focused synthetic input.
6. Extend the co-located `*.test.ts` assertions and snapshots; update public documentation and changelog if normalized behavior changes.

## Gotchas

- A user-role `tool_result` event is not a prompt and must not start a turn.
- One assistant response may span several JSONL lines with repeated provider usage; count it once by provider message id.
- Tool-result content may be a string or an array of parts.
- Malformed/unknown lines should become warnings or pass-through behavior, not crash the full session.
- File tool payload location differs by Read versus Write/Edit/MultiEdit; keep parser and fork-writer interpretations aligned.

## Verify

- [ ] `pnpm test -- packages/adapter-claude-code`
- [ ] `pnpm typecheck`
- [ ] `pnpm lint`
- [ ] Golden normalized model and targeted edge-case assertions pass.
- [ ] If raw content location changed, fork and validator tests still pass.

## Debug

Trace `buildBlocks` → `stitch` → file/memory lifting with `mex graph query what-calls`; compare the fixture's exact JSONL line with the normalized `Message`, `ToolCall`, and `FileOp`. If a turn count shifts, inspect prompt detection before changing `buildTurns`.

## Update Scaffold

- [ ] Update `.mex/ROUTER.md` "Current Project State" if what's working/not built has changed
- [ ] Update any `.mex/context/` files that are now out of date
- [ ] If this is a new task type without a pattern, create one in `.mex/patterns/` and add to `INDEX.md`

