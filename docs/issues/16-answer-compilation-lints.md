# Title

Answer hashing at compile time and spoiler lints

# Summary

Implement the compiler stage that converts authored answers into salted hashes (via the issue 10 module), generates the scenario salt, and runs the DS7xxx answer lints including the `spoilerGuard.decoys` collision check.

# Context

ADR-007 fixes the scheme; DESIGN §8.4 defines the lints that protect authors from the two silent failure modes: an answer that can never match (empty after normalization) and a decoy name that would *accidentally* match (an innocent character judged guilty).

# Scope

- `packages/compiler/src/answers.ts`:
  `compileAnswers(questions: QuestionsFile, render: RenderProseFn, opts: {salt?: string, questionsFilePath: string}): Promise<{salt: string, compiledQuestions: CompiledQuestion[], compiledFinal: CompiledQuestion, diagnostics: Diagnostic[]}>`
  — `QuestionsFile` is the schema-validated object from `LoadedScenario.questions` (issue 12); `RenderProseFn` is issue 13's `renderProse` pre-bound with the manifest's `allowExternalLinks`; DS7xxx diagnostics carry `file: questionsFilePath` (line numbers `null` — YAML-node mapping is not required here).
- Unit tests.

# Detailed Requirements

1. Salt: use `opts.salt` when provided — the CLI validates the flag format before calling (malformed `--salt` is a usage error, exit 2, per DESIGN §10.4); this function `assert`s `SALT_PATTERN` and throws a programmer error on violation (no DS code). Otherwise generate 16 random bytes via `globalThis.crypto.getRandomValues` (DESIGN §8.2), hex-encode.
2. Per `text` question: hash each accepted answer with `hashAnswer(salt, a)`; deduplicate **by normalized form, keeping first-occurrence order** — later duplicates emit DS7101 and are dropped.
3. Per `choice` question: `answerHashes = [await hashAnswer(salt, correctOption)]`; shipped `options` keep ids + labels only.
4. Lints (run before hashing, on normalized forms):
   - DS7003 (error): an accepted answer or `correctOption` normalizes to `""`.
   - DS7101 (warning): two accepted answers of the *same* question normalize identically (message lists both source strings).
   - DS7002 (error): any `spoilerGuard.decoys` entry normalizes equal to any accepted answer of **any** question incl. final (message names the decoy, the question id, and the colliding accepted string).
5. Compiled output must contain **no** `accepted`, `correctOption`, `spoilerGuard`, or other plaintext-answer-bearing field — enforce by constructing fresh objects (never spread the authored object) and by a test that walks the compiled JSON asserting the key blacklist.
6. Hint/explanation/prompt prose is rendered via the injected `render` function (issue 13): `promptHtml`, `explanationHtml`, `hints[].html` — DS5xxx diagnostics from rendering are propagated into this stage's `diagnostics`, and the compiled output contains only `*Html` fields, never raw `prompt`/`explanation`/`hints` strings.

# Acceptance Criteria

- [ ] With pinned salt `0123…def` (golden test salt), compiled hashes for the fixture equal the issue 10 vector expectations.
- [ ] DS7002/DS7003/DS7101 fixtures each produce exactly that code; DS5xxx from a hostile prompt fixture propagates through this stage.
- [ ] Key-blacklist walker test passes on `valid-full` compiled output (`accepted|correctOption|decoys|spoilerGuard` appear nowhere).
- [ ] Random-salt path produces 32-hex salt and differing hashes across two builds (test), while pinned-salt path is deterministic.
- [ ] Duplicate accepted answers dedupe to one hash.

# Validation

Unit tests per above; `valid-full` snapshot (pinned salt) covers integration. Issue 42's E2E proves player-side hash equality end-to-end.

# Dependencies

- 09, 10, 12, 13, 18

# Non-goals

- Judging UX and attempt handling (issue 36).
- Encrypted content gating (ADR-007 alternatives; v2).

# Design References

- DESIGN.md §8.2–§8.4, §6.2 (compiled question shape), §11.1 (DS7xxx) · ADR-007
