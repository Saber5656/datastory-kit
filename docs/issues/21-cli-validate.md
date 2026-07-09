# Title

datastory validate command

# Summary

Implement `datastory validate <dir>`: run the full compiler validation pipeline (load → schemas → prose → data → graph → answer lints) without writing anything, and render diagnostics in pretty or `--json` form with the §10.1 exit-code contract.

# Context

`validate` is the author's inner loop (DESIGN §2.2 step 3) and the machine interface for editors/CI/agents (§11.3). It calls issue 17's `compileScenario(dir, {validateOnly: true})` — the single public compile API — so this issue is wiring and contract, not new validation logic. Because it feeds untrusted packages into the compiler, it inherits trust-boundary-1 duties: it must go through the hardened loader (issue 12) only, perform no network access, and never execute or import author files.

# Scope

- `src/commands/validate.ts` in `packages/cli`; spawn-based tests.

# Detailed Requirements

1. Signature: `datastory validate <dir> [--json] [--max-warnings <n>]` (DESIGN §10.3).
2. Pipeline: `compileScenario(dir, {validateOnly: true})` — must execute *all* stages that can produce diagnostics, including Markdown rendering (DS5xxx) and CSV parsing/coercion (DS4xxx), but write no files and skip media copying.
3. Exit codes: any error-severity diagnostic → 1; warnings only → 0 unless `--max-warnings` exceeded → 1; `<dir>` missing/not a directory → 2 (usage); internal fault → 3.
4. Output: pretty reporter (stderr) by default; `--json` prints the §11.3 envelope to stdout (and nothing else to stdout). `--quiet` semantics (per DESIGN §10.1 "suppresses non-error output"): exit 0 → no output at all; exit 1 → error diagnostics + summary line only (warnings shown only when they caused the failure via `--max-warnings`).
5. Success message (pretty, non-quiet): exactly `✓ {title} ({id}) is valid — {chapters} chapters, {questions} questions, {documents} documents, {images} images, {tables} tables, {characters} characters` (counts include the final accusation in `questions`).
6. Performance guard: on the `valid-full` fixture, completes < 2 s (soft assertion in test with generous 10 s CI bound; documents the intent).

# Acceptance Criteria

- [ ] `validate` on `valid-full` fixture exits 0 with the inventory line.
- [ ] Each defect-fixture family (DS1/2/3/4/5/6/7 — reuse compiler fixtures) exits 1 and shows the expected code in pretty output.
- [ ] `--json` output on a defect fixture parses and matches the envelope schema; stdout contains *only* JSON (byte-level assert).
- [ ] `--max-warnings 0` turns a warning-only fixture into exit 1.
- [ ] Missing directory exits 2.
- [ ] Symlink-escape fixture and hostile Markdown/CSV fixtures (reused from issues 12–14) surface their DS codes through the CLI (boundary-1 wiring proof).
- [ ] No files are created/modified anywhere during validate — asserted by a recursive before/after tree hash of the scenario directory and the temp cwd in the test.

# Validation

Spawn-based CLI tests per above; contract snapshot of `--json` output; tree-hash no-write check as specified.

# Dependencies

- 17, 18, 19

# Non-goals

- Watch mode (v2); building output (issue 22).

# Design References

- DESIGN.md §10.3 (validate), §11.2–§11.3 (output formats), §10.1 (exit codes)
