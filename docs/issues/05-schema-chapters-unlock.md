# Title

Chapter schema and unlock-rule types

# Summary

Add the chapter frontmatter schema and the `Unlock` discriminated union (`start` / `solved` / `afterQuestions`) to `@datastory/schema`, in both authored form (YAML frontmatter values) and compiled form (the `{kind: …}` objects used in `story.json` and by the player's gating engine).

# Context

Chapters are the only unlock axis in v1 (DESIGN §3.2, §7). The authored form in frontmatter is `unlock: start`, `unlock: solved`, or `unlock: {afterQuestions: [...]}` (§5.4); the compiled form is a discriminated union (§6.2). Both the compiler (validation, graph checks in issue 15) and the player (gating engine, issue 25) consume these types, so they live in the schema package.

# Scope

- `packages/schema/src/chapter.ts` + `packages/schema/src/chapter.test.ts`, exported types.

# Detailed Requirements

1. `ChapterFrontmatterSchema` (`.strict()`):
   - `id`: IdSchema. `title`: string 1..120. `order`: int ≥ 1.
   - `unlock`: union of literal `"start"`, literal `"solved"`, object `{ afterQuestions: IdSchema[] }` with `afterQuestions` non-empty, max 10, unique items (canonical per DESIGN §5.4; the "must not contain the final accusation id" rule is cross-file and belongs to issue 15 / DS3010).
2. Compiled form types + schema (exported through the issue 11 barrel; the player's bundle loader, issue 24, consumes them from there):
   - `type Unlock = {kind:"start"} | {kind:"solved"} | {kind:"afterQuestions", questionIds: string[]}`.
   - `UnlockSchema` as a Zod discriminated union on `kind`.
   - Pure function `compileUnlock(authored): Unlock` exported here so compiler and tests share one mapping.
3. `CompiledChapterSchema`: `{ id, title, order, unlock: UnlockSchema, bodyHtml: string }` — used by issue 24's bundle validation.
4. Uniqueness of `id`/`order` across chapters is a *package-level* check (multiple files) — export helper `checkChapterSetInvariants(chapters: ChapterFrontmatter[]): {duplicateIds: string[], duplicateOrders: number[], startCount: number}` as a pure function; the compiler (issue 15) maps its output to DS3007/DS3001 diagnostics per DESIGN §7.3.
5. Markdown body content is **not** validated here (free text; rendering/sanitizing is issue 13).

# Acceptance Criteria

- [ ] Accepts all three §5.4 unlock forms; rejects `unlock: {afterQuestions: []}`, duplicated question ids in the list, unknown keys, `order: 0`, `order: 1.5`.
- [ ] `compileUnlock("start")` → `{kind:"start"}`; `compileUnlock({afterQuestions:["q-a"]})` → `{kind:"afterQuestions", questionIds:["q-a"]}` (field rename covered by test).
- [ ] `checkChapterSetInvariants` correctly reports duplicates and start-count on crafted fixtures (0, 1, 2 start chapters).
- [ ] Types exported from the package barrel; player-facing `Unlock` type has no Zod dependency leakage (plain type export).

# Validation

Table-driven unit tests per above; typecheck-level test that `Unlock` narrows correctly in a `switch` on `kind` (exhaustiveness via `never` check).

# Dependencies

- 04

# Non-goals

- Graph invariants (reachability, acyclicity, exactly-one-start as an *error*) — issue 15.
- Frontmatter extraction from Markdown files — issue 12/13.

# Design References

- DESIGN.md §5.4 (authored chapters), §6.2 (compiled shape), §7.1–§7.3 (semantics)
