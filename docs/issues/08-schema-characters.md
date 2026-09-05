# Title

Character and interview-topic schema

# Summary

Add the `characters/*.yaml` schema to `@datastory/schema`: character identity, optional portrait, and the topic list with sibling-only `requires` prerequisites — including the per-file acyclicity check of the requires graph.

# Context

DESIGN §5.7 defines interviews as informational-only content (no unlock side effects, §3.2). Each character file is self-contained: topics may only require sibling topics, so cycle detection and reference checks are *file-local* and belong in the schema (unlike cross-file checks, which are issue 15).

# Scope

- `src/character.ts` in `packages/schema`, exports, unit tests.

# Detailed Requirements

1. `TopicSchema` (`.strict()`): `id` IdSchema · `label` string 1..120 · `requires` array of IdSchema, max 5, unique, default `[]` · `reply` string 1..4000 (Markdown; rendered by issue 13). All caps canonical per DESIGN §5.7.
2. `CharacterFileSchema` (`.strict()`): `id` IdSchema · `name` string 1..80 · `role` string 1..60 · `portrait` RelPathSchema ending `.png|.jpg|.jpeg|.webp` optional · `unlockedBy` IdSchema · `topics` array of TopicSchema, 1..12.
   Refinements (each with a distinct error message):
   - topic ids unique within the file;
   - every `requires` entry references a sibling topic id (self-reference forbidden);
   - the requires digraph is acyclic — pure helper `findTopicCycle(topics): string[] | null` with a deterministic contract: DFS in authored topic order (edges in authored `requires` order); returns the first cycle found as a **closed path** (`["a","b","a"]`); a self-reference would return `["a","a"]` but is already rejected by the refinement above; returns `null` for a DAG.
3. Compiled form `CompiledCharacterSchema` (`.strict()`): `{id,name,role,portraitSrc?,unlockedBy,topics:[{id,label,requires,replyHtml}]}` — `portraitSrc` follows the same `media/` path rules as issue 06's `src`; `topics` 1..12; `replyHtml` is shape-validated as a string only (sanitization is compile-time, issue 13).
4. Export `findTopicCycle` (the player's interview UI, issue 35, reuses it in a dev assertion).

# Acceptance Criteria

- [ ] Accepts the DESIGN §5.7 example (two topics, one `requires` chain).
- [ ] Rejects: 13 topics, `requires: ["topic-nonexistent"]`, self-require, 2-cycle (`a→b→a`) and 3-cycle with the cycle path present in the error, duplicate topic ids, portrait `x.svg`, unknown keys.
- [ ] `findTopicCycle` returns `null` for a DAG and a concrete cycle array for cyclic fixtures.

# Validation

Table-driven unit tests; property-style test optional (random DAGs never report a cycle).

# Dependencies

- 04

# Non-goals

- Package-level cap "≤ 20 characters" and portrait file validation — issue 12 (loader).
- `unlockedBy` chapter existence — issue 15.
- Reply rendering — issue 13. Interview UI — issue 35.

# Design References

- DESIGN.md §5.7 (characters), §3.2 (informational-only interviews), §6.2 (compiled form)
