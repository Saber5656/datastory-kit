# Title

Question, hint, and final-accusation schema

# Summary

Add the `questions.yaml` schema to `@datastory/schema`: choice/text question variants with conditional fields, tiered hints, the mandatory single `finalAccusation`, and the optional `spoilerGuard` block.

# Context

DESIGN §5.8 fixes the authored shape; §8 consumes `accepted`/`correctOption` at compile time (issue 16) and drops them from the bundle (ADR-007). The schema must make invalid combinations (e.g. `accepted` on a choice question) unrepresentable at parse time, so weaker downstream code never branches on malformed data.

# Scope

- `src/question.ts` in `packages/schema`, exports, unit tests.

# Detailed Requirements

1. `HintSchema`: string 1..500 (Markdown; caps canonical per DESIGN §5.8). Max 5 hints per question (array bound).
2. `OptionSchema` (`.strict()`): `id` IdSchema · `label` string 1..120.
3. `QuestionSchema` = discriminated union on `type`:
   - Common fields: `id` IdSchema · `chapter` IdSchema · `prompt` string 1..1000 (Markdown) · `explanation` string ≤4000 optional (Markdown) · `hints` HintSchema[] max 5 default `[]`.
   - `type: "choice"`: `options` OptionSchema[] 2..6, unique option ids (labels need not be unique); `correctOption` IdSchema; refinement `correctOption ∈ options[].id`. Field `accepted` must be absent.
   - `type: "text"`: `accepted` array of string 1..120, 1..10 items, unique **as written** (normalized-duplicate detection is issue 16's DS7101 warning — DESIGN §5.8/§8.4). Fields `options`/`correctOption` must be absent.
4. `QuestionsFileSchema` (`.strict()`): `{ questions: QuestionSchema[], finalAccusation: QuestionSchema, spoilerGuard?: { decoys: string[] (1..20, each 1..120) } }`.
   Refinements: total question count (incl. final) ≤ 40; all question ids globally unique (incl. final); `questions` array may be empty only if at least the final accusation exists (min 0 gate questions is allowed — a one-chapter mystery).
5. Compiled forms per §6.2: `CompiledQuestionSchema` (`.strict()` — unknown keys rejected) `{id,chapter,type,promptHtml,options?,answerHashes,explanationHtml?,hints:[{html}]}` with `Sha256HexSchema = /^[a-f0-9]{64}$/` for every entry of `answerHashes`; refinement: `type:"choice"` requires exactly 1 hash and 2..6 options, `type:"text"` requires 1..10 hashes and no `options`. `correctOption`/`accepted`/`spoilerGuard` intentionally have **no** compiled counterpart — `.strict()` makes their presence a parse error.
6. Export inferred types via the barrel (all field caps canonical per DESIGN §5.8).

# Acceptance Criteria

- [ ] Accepts the full DESIGN §5.8 example.
- [ ] Rejects: choice with 1 or 7 options, `correctOption` not among options, choice with `accepted`, text with `options`, 11 accepted answers, duplicate accepted strings, 6 hints, hint of 501 chars, duplicate question ids across `questions` and `finalAccusation`, missing `finalAccusation`, 41 total questions, unknown keys.
- [ ] Type-level: `Question` narrows on `type` such that `q.accepted` only typechecks under `"text"`.
- [ ] `CompiledQuestionSchema` rejects objects carrying `accepted`, `correctOption`, or `spoilerGuard` keys (three explicit tests), rejects a 63-char or non-hex hash, and enforces the per-type hash-count refinement.

# Validation

Table-driven unit tests for every case above; exhaustive `switch` compile check on the union.

# Dependencies

- 04, 05 (chapter id type reuse)

# Non-goals

- Chapter-reference existence, reachability (issue 15); hashing and decoy lints (issue 16); judging UI (issue 36).

# Design References

- DESIGN.md §5.8 (authored), §6.2 (compiled), §8 (answer checking), ADR-007
