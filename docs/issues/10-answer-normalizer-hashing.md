# Title

Answer normalization and hashing module with golden vectors

# Summary

Implement `normalizeAnswer` and `hashAnswer` in `@datastory/schema` exactly per DESIGN §8.1–§8.2, backed by a committed golden-vector file that pins the behavior for both compiler (Node) and player (browser) forever.

# Context

This is the judging contract: if compiler and player ever disagree on one code point, learners get unwinnable questions. ADR-007 therefore mandates a *single* shared implementation with golden vectors. Node ≥ 20 exposes WebCrypto as `globalThis.crypto.subtle`, so one async implementation serves both environments with zero conditional code.

# Scope

- `packages/schema/src/answer.ts`, `packages/schema/src/answer.vectors.json`, `packages/schema/src/answer.test.ts`; `normalizeAnswer`, `hashAnswer`, `SALT_PATTERN` exported from `packages/schema/src/index.ts`.

# Detailed Requirements

1. `normalizeAnswer(s: string): string` — steps in exactly this order (DESIGN §8.1):
   1. `s.normalize("NFKC")`.
   2. Trim, then collapse every run of Unicode whitespace (`/\s+/gu` plus U+3000 — note NFKC already folds U+3000 to space, rely on `\s`) to a single ASCII space.
   3. `.toLowerCase()` (documented as sufficient for ja/en v1).
   4. Katakana→Hiragana: map code points U+30A1–U+30F6 by −0x60; leave U+30FC (ー) and everything else untouched.
   5. Strip *leading and trailing* characters from the set `。、．，「」『』"'` and ASCII `.` `,` `"` `'` (loop until stable).
2. `hashAnswer(salt: string, answer: string): Promise<string>` — `sha256hex(utf8(salt + ":" + normalizeAnswer(answer)))` using `globalThis.crypto.subtle.digest("SHA-256", …)`; lowercase hex output. Throw a descriptive error if `crypto.subtle` is unavailable.
3. `SALT_PATTERN = /^[0-9a-f]{32}$/` export (16-byte hex; generation happens in the compiler, issue 16).
4. Golden vectors: `packages/schema/src/answer.vectors.json`, committed, ≥ 20 entries of `{id, steps: string[], input, normalized, hashWithTestSalt}` where `steps` tags which §8.1 pipeline steps the vector exercises (`"nfkc"|"whitespace"|"casefold"|"kana"|"punct"`) and the test salt is the literal `0123456789abcdef0123456789abcdef`. A test asserts every step tag appears in ≥ 2 vectors (mechanical replacement for "review confirms"). Must include at minimum:
   - `"ＳＡＴＯ　Ｒiku"` (full-width alnum + ideographic space), `"ｻﾄｳ ﾘｸ"` (half-width katakana → NFKC → katakana → hiragana), `"サトウリク"` vs `"さとうりく"` (must normalize identically), `"  佐藤　 リク  "` (whitespace collapse), `"「佐藤リク」"` and `"佐藤リク。"` (punctuation strip), `"O'Brien"` (inner apostrophe **kept** — strip is edges-only), `"3人"` vs `"３人"`, `"カー"` (ー preserved → `かー`), `"ABC"` vs `"abc"`, empty-after-normalization case (`"。"` → `""`).
5. Tests iterate the vector file (no hardcoded expectations in test code) and additionally assert idempotence: `normalizeAnswer(normalizeAnswer(x)) === normalizeAnswer(x)` for all vectors.

# Acceptance Criteria

- [ ] All golden vectors pass on Node 20 and Node 22 (CI matrix).
- [ ] The steps-coverage test passes (every §8.1 step tagged in ≥ 2 vectors); browser re-verification is owned by issue 42 per ISSUE_PLAN §6-2.
- [ ] `hashAnswer` output for vector 1 manually cross-checked against an independent tool (e.g. `printf '%s' 'salt:normalized' | shasum -a 256`) and the check recorded in the PR description.
- [ ] Public API documented with JSDoc including the ADR-007 "casual spoiler protection only" caveat.

# Validation

Unit tests above. Issue 42's E2E later re-validates the same vectors *inside a real browser* (WebKit + Chromium) — note this forward pointer in a code comment.

# Dependencies

- 04

# Non-goals

- Salt generation and question compilation (issue 16); judging UX (issue 36); kanji→reading conversion (out of scope; authors list reading variants — DESIGN U3).

# Design References

- DESIGN.md §8.1–§8.4, §3.4 U3 · ADR-007
