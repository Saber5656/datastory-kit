# Title

End-to-end playthrough test suite

# Summary

Build the Playwright E2E suite per DESIGN §14: scripted full solves of both sample locales on Chromium and WebKit — covering the no-SQL path, wrong answers, hints, dialogue chains, reload persistence, reset, zero CSP violations, and zero external requests — plus the axe a11y smoke and CI wiring.

# Context

This is the product gate (ISSUE_PLAN §6-4): it proves the real compiled artifact of the real sample plays correctly in real browsers, closing the loop that unit layers can't (WebCrypto hashing on WebKit, wasm loading, localStorage, CSP interplay — DESIGN §3.4 U1/U2).

# Scope

- `e2e/` workspace package: Playwright config, fixtures orchestration (build samples via CLI, serve via `preview`), spec files, axe integration, CI job.

# Detailed Requirements

1. Setup: global-setup builds both sample packages with pinned `--salt/--built-at` via the real CLI, then starts `datastory preview` per bundle (distinct ports); teardown stops servers.
2. Projects: `chromium` and `webkit`, both samples → 4 project×locale combinations for the core solve spec **and** the SQL-path spec (both are §14-required coverage and run on the full matrix); only non-required helper/smoke specs may be Chromium-only (explicitly marked).
3. Core solve spec (per locale), driven by a shared solve-script module `e2e/solve-scripts/school-library-case.ts` exporting locale-parameterized steps (ids identical across locales — issue 41 isomorphism):
   - Title screen → start; read chapter 1; open a document (unread dot clears);
   - answer gate Q1 **wrong** once (retry copy), then correct → toast + new content appears;
   - interview: ask a `requires`-gated topic chain;
   - **no-SQL path**: solve Q2 using the table browser only (filters + sort);
   - SQL path (separate spec): solve Q2 alternatively via SQL console (typed query, results asserted);
   - open hint tier 1 on Q3 (confirmation flow), answer Q3;
   - **reload page** → all progress restored (chapter state, transcripts, hints);
   - final accusation: wrong once, then correct (kana-variant input for ja: `さとうりく`; order-swapped for en: `sato riku`) → solved panel → epilogue readable → ending summary numbers equal the solve-script module's exported expectations object `{chaptersCompleted, questionsAnswered, totalAttempts, hintsOpened}` (the script derives them from its own scripted actions — issue 39's formulas);
   - settings reset (typed confirmation) → title screen, storage key gone.
4. Security/privacy assertions active during the *entire* core spec (§13.3/§13.6): collect `securitypolicyviolation` events (must be 0); intercept all requests — any non-same-origin request fails the test; assert no cookies set.
5. A11y smoke: axe-core scan on shell/title, each modality view, question card, accusation, ending — no serious/critical violations (per §14).
6. Golden-vector browser check (closes issue 10's loop per ISSUE_PLAN §6-2): the spec file `e2e/specs/vector-parity.spec.ts` imports `packages/schema/src/answer.vectors.json`, takes the **first 3 vectors by `id` order**, and for each runs `page.evaluate` computing `sha256hex(salt + ":" + normalized)` with page-context `crypto.subtle` (formula inline; salt = the vectors' fixed test salt), asserting equality with `hashWithTestSalt` — this validates WebCrypto behavior in the real browser without needing app internals.
7. CI tiering per DESIGN §14: the **PR gate** runs the full suite (core solve + SQL path + security assertions + axe) on Chromium for both locales; the WebKit matrix runs on `main` pushes and nightly `schedule` and is a release gate. New job in the CI workflow (needs `build`), Playwright browsers cached, traces uploaded on failure.
8. Flake policy: no arbitrary sleeps; use Playwright auto-wait + explicit `aria`-based locators; retries=1 on CI with trace-on-retry.

# Acceptance Criteria

- [ ] Core solve passes for ja+en on Chromium+WebKit locally and in CI (main).
- [ ] CSP-violation and external-request assertions demonstrably work (scratch: inject a violation, watch it fail, revert).
- [ ] Reload-persistence and reset steps assert storage contents (`datastory:v1:*` key present/absent).
- [ ] SQL-path spec runs a real query through the wasm engine and asserts a result row.
- [ ] Axe smoke green on all listed surfaces.
- [ ] Vector parity check green on WebKit.
- [ ] Total PR-mode E2E wall time ≤ 10 min.

# Validation

Run `pnpm --filter e2e test` locally (Chromium tier) and on a scratch branch verify three deliberate sabotages each fail the right spec, then revert: (a) edit an accepted answer in the ja sample → solve spec fails at that gate; (b) inject an inline `<script>` into the built index.html → CSP assertion fails; (c) add a `fetch("https://example.com")` to a fixture page eval → external-request assertion fails. Record the three failing runs in the PR.

# Dependencies

- 22, 23, 29–35, 38, 39, 40, 41

# Non-goals

- Visual-regression screenshots (v2); load/perf testing beyond the §15 informational Lighthouse note; Firefox project (add post-v1 if demand).

# Design References

- DESIGN.md §14 (E2E row), §16.3-1 (no-SQL path), §13.3/§13.6 (assertions), §3.4 U1/U2
