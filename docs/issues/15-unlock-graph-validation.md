# Title

Unlock-graph compilation and invariant checks

# Summary

Implement the cross-file validation stage: resolve every id reference, then verify the chapter-unlock graph invariants of DESIGN §7.3 (exactly one start, acyclic, reachable, no self-locks) producing DS3xxx diagnostics.

# Context

These checks are what make authored packages *playable by construction*: any scenario that compiles cannot soft-lock a learner (content unreachable, circular gates). The gating engine (issue 25) is deliberately dumb — it trusts these invariants — so this stage must be exhaustive.

# Scope

- `packages/compiler/src/graph.ts` exporting one entry point:
  `validateGraph(loaded: LoadedScenario): { diagnostics: Diagnostic[]; reachableChapters: Set<string> | null }` — internally split into reference resolution and graph invariants; `reachableChapters` is non-null only when no error-severity diagnostic was produced.
- Unit tests + fixtures under `packages/compiler/test/`.

# Detailed Requirements

All invariant numbers and DS codes below are canonical per DESIGN §7.3 (items 1–10).

1. Reference resolution (DS3002, one diagnostic per broken ref, message names the referrer and the missing id):
   - chapter `unlock.afterQuestions[*]` → existing **gate** question id; referencing the final accusation's id → DS3010 (DESIGN §5.4/§7.3-10: the final sets `solved`, never `correct`, so such a gate could never open).
   - every `unlockedBy` (documents, images, tables, characters) → existing chapter id.
   - every question `chapter` (incl. final) → existing chapter id.
2. Chapter-set invariants via issue 05's `checkChapterSetInvariants`: duplicate chapter ids/orders → DS3007; `startCount != 1` → DS3001 (message states found count).
3. Graph construction: nodes = chapters; directed edge `A → B` iff B's `afterQuestions` contains a question whose `chapter == A.id`. Questions on `solved` chapters may not appear in any `afterQuestions` → DS3008 (post-solve questions cannot gate pre-solve content).
4. Invariants:
   - Cycle detected → DS3003 listing one cycle path (`ch-a → ch-b → ch-a`).
   - Non-`solved` chapter unreachable from start → DS3009 (one diagnostic per chapter).
   - Self-lock (question's own chapter transitively depends on that question) → DS3004.
   - `finalAccusation.chapter` unreachable → DS3005.
   - `unlockedBy` pointing at an unreachable chapter → DS3006 (content could never appear).
   - No `solved` chapter exists → warning DS3101 (epilogue recommended).
5. Suppression rule (mechanical): when DS3002/DS3010 fired for a reference, skip exactly the checks that traverse that edge — (a) reachability/cycle analysis treats the broken `afterQuestions` entry as absent, and (b) DS3006 is skipped for content whose `unlockedBy` itself failed resolution. All other diagnostics still run; every suppression site carries a comment naming this rule.
6. `reachableChapters` is reused by the emitter (17) and tests.

# Acceptance Criteria

- [ ] `valid-full` fixture passes with zero DS3xxx.
- [ ] One fixture per code DS3001–DS3010 + DS3101 produces exactly that code (and no cascade spam — asserted diagnostic counts).
- [ ] Cycle fixture message contains the full cycle path in order.
- [ ] Two-start-chapters fixture reports DS3001 once (not per chapter).
- [ ] Property test (fast-check or hand-rolled, 200 random DAG scenarios): generated valid graphs never produce errors; generated single-defect graphs produce the matching code.

# Validation

Fixture + property tests per above.

# Dependencies

- 05, 09, 12, 18

# Non-goals

- Runtime unlock evaluation (issue 25 — shares semantics via DESIGN §7, not code).
- Answer/decoy lints (issue 16).
- Pedagogical lints like orphan-content warnings beyond DS3006 (manual review per DESIGN §16.3).

# Design References

- DESIGN.md §7.3 (invariants), §5.4 (unlock forms), §11.1 (DS3xxx)
