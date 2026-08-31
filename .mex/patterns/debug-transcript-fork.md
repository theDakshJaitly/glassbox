---
name: debug-transcript-fork
description: Diagnose planned evictions that are not found, structural validation failures, or forks that fail to re-parse/resume.
triggers:
  - "fork failed"
  - "notFound eviction"
  - "orphan tool"
  - "failed to re-parse"
edges:
  - target: context/cleaning.md
    condition: always load for cleaning invariants and write gates
  - target: patterns/change-cleaning-pipeline.md
    condition: when the diagnosis requires a behavior change
  - target: patterns/add-transcript-parsing.md
    condition: when parser and fork-writer disagree about raw payload location
grounds_to:
  - node: "function:8a8fee82b46da0a3d691d4913f234067"
    fingerprint: "mh:64:7b226d696e68617368223a5b373433353431352c37333831363733372c373433353332362c32373933343839392c34373637333131372c35383939343134302c323731343330352c34343739383935302c38353839303133372c36323236373432352c34303430323137312c31363338303535302c36373436383937392c34323033363635302c32353330363039302c35393637383033382c31333336363334362c32353233373034342c36303934393036332c32353534313330372c35313534373633392c32343833313333342c363838333432392c36383031393139322c31363032323230352c31383531363231312c31343634383534392c31353630373931342c3239313133352c31303332333734382c31333539343332332c37343239303535372c31373536313039372c363938343133322c32313536363834332c31383438363434382c35323933373735322c32393733303432312c393139313330332c32343730323836392c34383937383139382c31353539383738382c36333434333935312c32313037393237362c31353734393034352c34303937323631342c33353034363131372c34313635363636302c38383539323736322c3131333132303036392c343538383539332c34363338343235342c3737353231392c31383234373139322c34393738303838392c373936373930382c35323732302c31323139363435312c37373132363831312c32323931323839392c32303636333635332c31343233393535322c31363336383636352c33353138313433355d2c226e65696768626f7273223a5b2266756e6374696f6e3a3138326435313165303332303631633933313837353262613365386539353630222c2266756e6374696f6e3a3465303637373635366234346361323265356538343636376239316465376563222c2266756e6374696f6e3a3561623130653462323938393663366230306135376163326432333765353966222c2266756e6374696f6e3a3961393234666638353535343838346533383232366531333534663465363632222c2266756e6374696f6e3a6231373834383331626137336331396332616562653961303031313438303763225d2c22746f6b656e436f756e74223a3236307d"
  - node: "function:ebc4d10f43d3fe62e54f075a990c6764"
    fingerprint: "mh:64:7b226d696e68617368223a5b373433353431352c36383630383839382c33353532343933372c38393232373739362c3134353838323135312c3133393437323634342c323731343330352c3138353136313834382c3334383233303338352c3230343131333937392c31323737373330382c3232333239313239362c36373436383937392c34393439313531312c3237333939363838342c32353932313238392c3133363635363236332c3238393435353130392c3233383836333230352c3234313939302c3139313338353035352c3136363531343533342c31333237323838382c32373236393136392c38373234323032382c31393838393332372c31343634383534392c36313937383432352c3432333534393237392c31303332333734382c37303130313637302c37343239303535372c31373536313039372c363938343133322c34313937383934302c34363136313136392c3235323238323230392c38373437363636392c31333432383435382c38303730373334302c36303334363735342c35313437353533322c36343333343132312c32313037393237362c3231343339323538312c36393336313532312c3134303038393531322c34313635363636302c37323035313131342c3131353138363736352c393934393334312c34363338343235342c3737353231392c31383234373139322c3138343539303636352c3130353330383037312c35323732302c34383834353436352c35313733373930392c36383933323333302c3338303932383230362c36393632313434332c3131383936363032372c3239313633363735345d2c226e65696768626f7273223a5b2266756e6374696f6e3a6231373834383331626137336331396332616562653961303031313438303763225d2c22746f6b656e436f756e74223a36317d"
last_updated: 2026-08-31
---

# Debug Transcript Forks

## Context

[`validateTranscript()`](mex://function:8a8fee82b46da0a3d691d4913f234067) inventories JSON, message content, UUID lineage, and tool pairs. [`newProblems()`](mex://function:ebc4d10f43d3fe62e54f075a990c6764) filters out problems already present in the source, so only regressions block a fork.

## Steps

1. Preserve the original and failing derived text; reproduce in memory before any write.
2. Identify the boundary:
   - eviction action absent/`notFound` → analyzer origin or raw payload lookup;
   - `newProblems` non-empty → transformation broke structure/content;
   - validator passes but adapter parse fails → validator/parser coverage mismatch;
   - parse succeeds but resume fails → session id/filename or unsupported Claude invariant.
3. For `notFound`, trace `segmentId → originToolCallId → raw tool_use/tool_result id`; inspect Read result, Write/Edit input, and top-level mirror locations.
4. Run validation on source and fork and compare exact `{code, ref}` entries; do not “fix” a problem already present in both.
5. Confirm every written line that had a `sessionId` was restamped and the output filename equals the new id.
6. Add the minimal regression fixture/test at the failing boundary before changing behavior.

## Gotchas

- Pre-existing malformed lines or orphan pairs are allowed only if the fork introduces no additional problem with the same code/reference model.
- Replacing content with an empty string creates a validation failure; tombstones must stay non-empty.
- A mirrored `toolUseResult` can preserve heavy bytes even when the API-facing block is correct.
- Post-write re-parse failure occurs after a file exists; report it as unsafe and never suggest resuming it.
- Do not relax validation merely to make one fixture pass; first prove the source already has the same issue or expand the transformer.

## Verify

- [ ] Reproduction fails before the fix and passes afterward.
- [ ] `newProblems(validateTranscript(source), validateTranscript(fork))` is empty.
- [ ] Planned action count, fork `evicted`, and `notFound` agree.
- [ ] Fork parses into the expected message/tool/file-op counts.
- [ ] Source bytes are unchanged; output id and filename match.
- [ ] Adapter fork/validate tests plus `pnpm typecheck` and `pnpm lint` pass.

## Debug

If the validator and parser disagree, resolve both symbols with the graph and compare their accepted tree event/content rules. If only Claude resume disagrees, capture an anonymized minimal JSONL and add a validator rule that represents the demonstrated protocol constraint.

## Update Scaffold

- [ ] Update `.mex/ROUTER.md` "Current Project State" if what's working/not built has changed
- [ ] Update any `.mex/context/` files that are now out of date
- [ ] If this is a new task type without a pattern, create one in `.mex/patterns/` and add to `INDEX.md`

