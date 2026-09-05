# Title

Sample scenario (Japanese): The Missing First Edition (消えた初版本)

# Summary

Author the complete Japanese sample scenario per DESIGN §16 at `examples/school-library-case-ja/`: 4 chapters, 6 documents, 3 images, 4 data tables, 3 characters, 3 gate questions + final accusation with tiered hints — satisfying every pedagogical invariant of §16.3.

# Context

The sample is simultaneously: the flagship demo, the format's living documentation, the compiler/E2E fixture, and the template source for `init --template library-case` (issue 20). Its quality ceiling is the product's perceived quality ceiling. Non-violent premise fixed by the user decision (school audience).

# Scope

- Full scenario package at `examples/school-library-case-ja/` per §5.1; placeholder-free.
- Image assets: 3 simple original illustrations/diagrams (hand-drawn-style floor plan, shelf photo substitute illustration, bookend close-up) — original works, no stock/AI-ambiguous licensing; keep each ≤ 300 KB.
- Sync pass over issue 20's `library-case` template (`TODO(sample-sync)` markers).

# Detailed Requirements

1. Content inventory **exactly** per DESIGN §16.2 — the stated counts are the spec, not minimums; do not add extra chapters/evidence/tables/characters/questions. Premise per §16.1: sympathetic culprit (佐藤リク), redeemable motive (surprise restoration), resolution emphasizing evidence-based reasoning.
2. Mystery logic must be **deductively sound**: a solver document (`docs/research/sample-case-logic.md` in the *repo*, not the package) maps each question to the minimal evidence set proving it, and the accusation to the full chain — reviewers verify against it that no contradiction exists and that **each non-culprit is positively excluded by evidence** (DS7002 only catches name collisions, not logic holes — the solver doc carries this proof). `spoilerGuard.decoys` lists every plausible **non-culprit** answer/name variant; the culprit and all accepted answer variants must not appear in it (they would trigger DS7002 by design).
3. Pedagogical invariants (§16.3) each verified in review: no-SQL solvability per gate; no orphan content; text answers list variants (space/no-space, kana; e.g. `佐藤リク / 佐藤 リク / さとうりく`); ja reading level = upper-elementary with `<ruby>` furigana on above-level kanji; hints per question: nudge → method → near-answer. Images must contain **no baked-in text** (they are shared with the en package, issue 41).
4. Data tables (~200 rows total per §16.2) must be internally consistent (entry_log times vs club_schedule vs testimonies) — consistency asserted by a small standalone check script `examples/school-library-case-ja/checks.test.ts` (workspace test) querying the built case.db for the key alibi joins.
5. `scenario.yaml` complete minimum: `formatVersion: 1`, `id: school-library-case-ja`, `locale: ja`, `title: 消えた初版本`, `version: 1.0.0`, `description` (1–2 sentences), honest `estimatedPlayMinutes`, `contentWarnings: []`, `settings.allowExternalLinks: false` explicit, SQL settings omitted (defaults).
6. Package validates with **zero warnings** (`datastory validate` exit 0, no DS-codes at all).
7. LICENSE note: sample content (text + images) released under the repo MIT license; stated in the package `README-first.md` equivalent (a short `NOTES.md` allowed outside the schema-validated set? — **No**: keep the package layout schema-pure; put content licensing in the repo-level `examples/README.md`).

# Acceptance Criteria

- [ ] `datastory validate examples/school-library-case-ja` → exit 0, zero diagnostics (errors *and* warnings).
- [ ] `datastory build examples/school-library-case-ja -o .tmp/sample-ja-dist` succeeds; bundle layout matches §6.1.
- [ ] Logic solver doc `docs/research/sample-case-logic.md` committed and reviewed: every question provable from listed evidence, each non-culprit positively excluded, decoy list contains no accepted-answer variant; compile is DS7002-clean.
- [ ] On-paper no-SQL solve path documented in the solver doc (which table-browser filters/documents answer each gate) — the runtime no-SQL playthrough is issue 42's gate.
- [ ] Consistency test passes: `pnpm --filter examples test -- checks` (or the workspace-equivalent exact command recorded in the file header) against the built case.db.
- [ ] Furigana present on above-elementary kanji in chapters (reviewer spot-check); issue 20's `library-case` ja template synced (`TODO(sample-sync)` markers gone).
- [ ] All images original, text-free, ≤ 300 KB each, with meaningful `alt`.

# Validation

Exact commands: `datastory validate examples/school-library-case-ja` · `datastory build examples/school-library-case-ja -o .tmp/sample-ja-dist` · the consistency test command above · solver-doc peer review of narrative tone for the school audience. Full interactive playthrough (incl. the 45–90 min fresh-tester run) is executed under issue 42 once the player is complete; this issue's gate is authored-content correctness.

# Dependencies

- 21, 22 (toolchain to validate/build; content authoring can start once 21 exists)

# Non-goals

- English version (41); extra difficulty levels or bonus cases (v2); voice/audio (v2).

# Design References

- DESIGN.md §16 (full spec), §5 (format), §8.4/§5.8 (decoys), §3.4 U3 (name variants)
