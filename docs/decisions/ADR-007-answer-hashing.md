# ADR-007: Answer checking via salted SHA-256 over normalized answers (casual spoiler protection)

- Status: Accepted
- Date: 2026-07-10
- Deciders: Design agent (conservative default consistent with ADR-001)

## Context

Judging is client-side (ADR-001), so correct answers must ship inside the bundle in some form. Learners are students; the goal is to keep answers out of casual view (view-source, DevTools network tab, ctrl+F in `story.json`), while accepting that client-side secrets can never resist a determined attacker. Text answers also need tolerant matching (Japanese input has width/kana/case variation).

## Decision

1. `story.json` stores, per question, only: `answerHashes: string[]` (lowercase hex SHA-256), plus matching metadata. Plaintext accepted answers exist only in the authored `questions.yaml`.
2. **Hash input** = `scenarioSalt + ":" + normalize(answer)` where `scenarioSalt` is a compiler-generated random 16-byte hex string stored in `story.json` (it prevents cross-scenario rainbow reuse, nothing more).
3. **Normalization pipeline** (exact order, applied identically in compiler and player — single shared implementation in `@datastory/schema`):
   1. Unicode NFKC normalization (folds full-width/half-width forms).
   2. Trim leading/trailing whitespace; collapse internal whitespace runs to a single ASCII space.
   3. Unicode default case folding (lowercase for ASCII/Latin).
   4. Katakana → Hiragana folding (U+30A1–U+30F6 mapped down by 0x60; prolonged sound mark U+30FC preserved).
   5. Remove the characters `。、．，「」『』"'` and ASCII `.` `,` `"` `'` when they are leading/trailing.
4. **Choice questions**: the correct `optionId` is hashed with the same scheme (uniform code path).
5. Hashing uses the Web Crypto API (`crypto.subtle.digest`) in the player and `node:crypto` in the compiler, verified equivalent by shared golden test vectors.
6. **Documented residual risk**: small answer spaces (esp. multiple choice) are trivially brute-forceable offline. This is *casual spoiler protection, not a security control*. The authoring guide states this plainly; classroom-integrity concerns are out of scope for the tool.

## Consequences

Positive:

- No answer strings greppable in shipped artifacts; casual spoiling requires deliberate effort.
- One shared normalization module eliminates compiler/player judging drift (the classic failure mode of dual implementations).
- Tolerant matching handles Japanese input realities (ｷﾀﾑﾗ / キタムラ / きたむら all match).

Negative:

- Determined learners can brute-force; accepted and documented.
- Overly aggressive normalization could conflate distinct answers; mitigated by a compile-time lint that errors when two *different* accepted answers of the *same question set* normalize to identical strings, and warns on collisions across distractor names listed in `spoilerGuard.decoys` (DESIGN.md §5.7).

## Alternatives considered

- **Plaintext answers in bundle**: trivially spoiled by ctrl+F; rejected.
- **Encrypting story branches with the answer as key** (à la "cryptographic gating"): elegant for hard gates but breaks tolerant matching (any normalization variant changes the key) and complicates authoring; revisit in v2 for optional "sealed epilogue" content.
- **Server-side judging**: violates ADR-001.
