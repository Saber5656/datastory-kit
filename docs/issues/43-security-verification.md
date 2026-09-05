# Title

Security verification suite and SECURITY.md completion

# Summary

Mechanically verify the DESIGN §13 security model: the expanded sanitizer XSS corpus, CSP conformance checks on built output, plaintext-answer leak scanning, privacy assertions, and the completed SECURITY.md with threat model and data inventory.

# Context

Issues 12/13/22 *implement* controls; this issue proves them and pins them against regression. It is deliberately late (post-E2E) so its checks run against the real assembled product, and it produces the documentation a school IT reviewer or OSS security researcher will actually read.

# Scope

- `packages/compiler/test/xss-corpus/` expansion; `e2e/specs/security.spec.ts`; `scripts/scan-bundle.mjs`; SECURITY.md completion; threat-model doc polish.

# Detailed Requirements

1. XSS corpus expansion (≥ 40 cases; sources: OWASP XSS cheat-sheet patterns adapted): script/style/iframe/object/embed/svg/math, event handlers (onerror/onload/onfocus/autofocus tricks), `javascript:`/`data:`/`vbscript:` URLs incl. whitespace/entity obfuscation (`java\tscript:`, `&#106;avascript:`), meta-refresh, form/action, base-tag, mXSS-style nested cases (`<noscript><p title="</noscript><img src=x onerror=alert(1)>">`). Each case: input → sanitized output snapshot + assertion that output parses with zero disallowed nodes (walk with a DOM parser, don't just string-match).
2. Sanitizer differential guard: corpus runs against every prose entry point (chapter body, doc body, reply, prompt, explanation, hint) via the public compile API — proving no call site bypasses `renderProse` (complements the ESLint restriction of issue 13).
3. Bundle scan script (`scan-bundle.mjs`, runs in CI after sample builds): walks `dist/` asserting — no plaintext **accepted answers or `correctOption` values** from the source packages (reads the *source* questions.yaml; greps raw + normalized forms; decoys are *not* scanned — they are legitimate story content by design); `spoilerGuard`/`accepted`/`correctOption` keys absent from `story.json` (structural walk); no `<script` outside `assets/*.js` and `index.html` src tags; CSP meta present and byte-exact per §13.3; no inline event handlers in any html; no external URLs in html/js except allowlisted (`https://github.com/...` repo link in about dialog — maintain explicit allowlist).
4. E2E security specs (extend issue 42 infra):
   - the **built ja/en samples** already complete full playthroughs with zero CSP violations under issue 42 — this issue adds an explicit cross-reference assertion so the §13.3 requirement is traceable here;
   - a purpose-built hostile-content package `examples/security-fixture` (kept out of npm/template paths) exercised in **two builds**: one with `settings.allowExternalLinks: false` asserting links unwrapped + DS5101 at compile, one with `true` asserting `https:`/`mailto:` links carry `target="_blank" rel="noopener noreferrer"` and forbidden protocols are unwrapped + DS5102 — plus zero CSP violations and zero external requests at runtime for both.
5. Sanitizer differential guard covers **all** prose entry points explicitly including the final accusation: chapter body, doc body, reply, and `questions[]` *and* `finalAccusation` prompt/explanation/hints.
6. SECURITY.md completion (this file is the threat-model document — no separate doc): threat model summary table (§13.1), data inventory (exact localStorage schema §7.4, "no cookies/telemetry/network"), answer-hash residual-risk statement (ADR-007), no-CSV-export note (spreadsheet-injection class absent), scope exclusions (host integrity, classroom sharing), reporting flow (from issue 03), supported-versions table.
7. Add `docs/research/security-review-checklist.md`: the manual pre-release checklist (deps audit clean, scan-bundle green, corpus green, E2E security specs green, SECURITY.md current) referenced by the release issue 46.

# Acceptance Criteria

- [ ] Corpus: all ≥40 cases neutralized at every prose entry point incl. `finalAccusation` fields; DOM-walk assertion (not string-match) in place.
- [ ] `scan-bundle.mjs` green on both samples; deliberately planting an accepted answer in a fixture doc makes it fail (scratch-verified); `spoilerGuard` key-absence walk in place.
- [ ] Hostile-fixture E2E: both link-policy builds behave per §13.2; zero CSP violations; zero external requests.
- [ ] SECURITY.md complete per requirement 6 (reviewed against §13 section-by-section).
- [ ] CI: corpus + scan-bundle + the hostile-fixture security spec (Chromium) all run in the **PR gate** (consistent with DESIGN §14 tiering); WebKit repetition rides issue 42's main/nightly matrix.

# Validation

Scratch-sabotage tests for each guard (plant → fail → revert) recorded in the PR; external-reader review of SECURITY.md for clarity.

# Dependencies

- 13, 22, 42

# Non-goals

- Penetration testing of hosting platforms; cryptographic hardening of answers beyond ADR-007 (documented residual risk); dependency CVE monitoring process (issue 03 owns).

# Design References

- DESIGN.md §13 (entire), §7.4 (data inventory), §14 (security rows) · ADR-007
