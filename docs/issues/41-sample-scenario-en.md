# Title

Sample scenario (English adaptation): The Missing First Edition

# Summary

Produce the English adaptation of the sample at `examples/school-library-case-en/` per ADR-006 and DESIGN §16.3-5: a localized sibling package (`id: school-library-case-en`, `locale: en`) with romanized names, adapted cultural references, and identical mystery logic.

# Context

The user chose bilingual v1 samples. ADR-006 fixes the mechanism: a separate single-locale package, not inline translation. "Adaptation, not literal translation" (§16.3-5): the deduction chain and data must stay isomorphic so both packages share the E2E solve script skeleton (issue 42), but prose should read natively.

# Scope

- Full package `examples/school-library-case-en/`; shared images copied from the ja package (text-free illustrations by design — issue 40 must keep image text out; coordinate).

# Detailed Requirements

1. Structural isomorphism with the ja package — ids are **identical across both packages except `scenario.id`** (no id mapping layer). The workspace test `examples/isomorphism.test.ts` compares, and on failure prints a diff listing each mismatch: `chapters[].{id,order,unlock}`; document/image id sets + `unlockedBy`; table names + full column schemas; character ids, topic ids, `requires` edges; question ids/types/`chapter` links; final-accusation shape; per-question hint counts. Run: `pnpm --filter examples test -- isomorphism`.
2. Names romanized consistently (e.g. 佐藤リク → Riku Sato) with a mapping table committed to `docs/research/sample-case-logic.md` (extends issue 40's solver doc); data tables re-authored with romanized values so SQL/table-browser results read naturally in en (evidentiary facts — times, counts, alibis — identical to ja).
3. Accepted answers cover en input realities: `["Riku Sato", "riku sato", "Sato Riku"]` (order-swapped names — §8 normalization handles case/width, not order; list both orders explicitly).
4. Prose register: CEFR-B1-ish (§16.3-4), no ruby (en), school-appropriate tone preserved; cultural adaptation where needed (club activities phrasing etc.) without changing evidentiary facts (times, counts, alibis stay identical to ja).
5. `estimatedPlayMinutes` may differ; `contentWarnings: []`.
6. Validates with zero warnings; issue 40's consistency check is parameterized into `examples/sample-case-consistency.test.ts` running against **both** packages' built case.db files (fixed output dirs `.tmp/sample-ja-dist`, `.tmp/sample-en-dist`); run: `pnpm --filter examples test -- sample-case-consistency`.

# Acceptance Criteria

- [ ] `datastory validate examples/school-library-case-en` → exit 0, zero diagnostics.
- [ ] Isomorphism test green (and demonstrably fails with a readable diff when an id is renamed in a scratch check, reverted).
- [ ] `accepted` lists cover both name orders (`Riku Sato` / `Sato Riku`) and lowercase forms — authored-content assertion here; the runtime judging of these variants is exercised by issue 42's E2E (42 depends on this issue, not vice versa).
- [ ] Parameterized consistency test green for the en package.
- [ ] English review checklist entry in the PR: reviewer handle + confirmation of CEFR-B1-ish reading level, school-appropriate tone, adapted cultural references.
- [ ] Shared images contain no baked-in Japanese text (checked; else issue 40 regression).

# Validation

`datastory validate examples/school-library-case-en` · `datastory build examples/school-library-case-en -o .tmp/sample-en-dist` · isomorphism + parameterized consistency tests (commands above) · English review checklist. Runtime playthrough belongs to issue 42, where this package becomes the second E2E fixture.

# Dependencies

- 40

# Non-goals

- Additional locales (v2); machine-translation tooling (v2 translation-sync idea).

# Design References

- DESIGN.md §16.3-5 (adaptation rule), §8 (normalization limits → explicit name-order variants) · ADR-006
