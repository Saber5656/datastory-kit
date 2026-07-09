# datastory-kit — v1 Issue Plan

Status: Draft for review · Date: 2026-07-10
Derives from [DESIGN.md](./DESIGN.md). GitHub Issues derive from this plan and [issues/](./issues/); if they disagree, this repository's docs win and the GitHub side is stale.

## 1. v1 completion statement

> v1 is complete when **all 46 issues below are closed and validated**. At that point: an author can `init`/`validate`/`build`/`preview` a scenario package per DESIGN §5/§10; the compiled bundle plays fully offline in a browser with all five investigation modes, gated chapters, hints, and a final accusation (DESIGN §7–§9); the player chrome works in Japanese and English; the sample scenario ships in both locales and passes scripted E2E playthroughs including the no-SQL path; the security model of DESIGN §13 is implemented and mechanically verified; and all packages are publishable to npm from CI with provenance. Only newly discovered implementation unknowns (DESIGN §3.4) may add issues beyond this list.

No product behavior lives outside this plan: every DESIGN.md requirement maps to at least one issue (§5 coverage table).

## 2. Issue list in recommended execution order

| # | Issue file | Title (GitHub) | Wave |
|---|---|---|---|
| 01 | [01-monorepo-scaffold.md](./issues/01-monorepo-scaffold.md) | Monorepo scaffold and tooling baseline | 0 |
| 02 | [02-ci-pipeline.md](./issues/02-ci-pipeline.md) | CI pipeline: lint, typecheck, test, build | 0 |
| 03 | [03-repo-security-baseline.md](./issues/03-repo-security-baseline.md) | Repository security baseline (Dependabot, CodeQL, pinned actions, audit gate) | 0 |
| 04 | [04-schema-package-manifest.md](./issues/04-schema-package-manifest.md) | @datastory/schema scaffold and scenario manifest schema | 1 |
| 05 | [05-schema-chapters-unlock.md](./issues/05-schema-chapters-unlock.md) | Chapter schema and unlock-rule types | 1 |
| 06 | [06-schema-evidence.md](./issues/06-schema-evidence.md) | Evidence schemas: documents and images | 1 |
| 07 | [07-schema-datapack.md](./issues/07-schema-datapack.md) | Datapack schema: tables, columns, display config | 1 |
| 08 | [08-schema-characters.md](./issues/08-schema-characters.md) | Character and interview-topic schema | 1 |
| 09 | [09-schema-questions.md](./issues/09-schema-questions.md) | Question, hint, and final-accusation schema | 1 |
| 10 | [10-answer-normalizer-hashing.md](./issues/10-answer-normalizer-hashing.md) | Answer normalization and hashing module with golden vectors | 1 |
| 11 | [11-diagnostics-model-json-schema.md](./issues/11-diagnostics-model-json-schema.md) | Diagnostic model and JSON Schema export | 1 |
| 12 | [12-compiler-loader.md](./issues/12-compiler-loader.md) | Compiler scaffold and hardened scenario-package loader | 2 |
| 13 | [13-markdown-sanitize-pipeline.md](./issues/13-markdown-sanitize-pipeline.md) | Markdown rendering and sanitization pipeline | 2 |
| 14 | [14-csv-sqlite-builder.md](./issues/14-csv-sqlite-builder.md) | CSV to SQLite datapack builder | 2 |
| 15 | [15-unlock-graph-validation.md](./issues/15-unlock-graph-validation.md) | Unlock-graph compilation and invariant checks | 2 |
| 16 | [16-answer-compilation-lints.md](./issues/16-answer-compilation-lints.md) | Answer hashing at compile time and spoiler lints | 2 |
| 17 | [17-bundle-emitter.md](./issues/17-bundle-emitter.md) | Bundle emitter: story.json, media, case.db | 2 |
| 18 | [18-diagnostics-catalog-reporters.md](./issues/18-diagnostics-catalog-reporters.md) | Diagnostics catalog and pretty/JSON reporters | 2 |
| 19 | [19-cli-scaffold.md](./issues/19-cli-scaffold.md) | CLI scaffold: argument parsing, exit codes, output conventions | 3 |
| 20 | [20-cli-init.md](./issues/20-cli-init.md) | datastory init command and starter templates | 3 |
| 21 | [21-cli-validate.md](./issues/21-cli-validate.md) | datastory validate command | 3 |
| 22 | [22-cli-build.md](./issues/22-cli-build.md) | datastory build command with CSP injection | 3* |
| 23 | [23-cli-preview.md](./issues/23-cli-preview.md) | datastory preview static server | 3 |
| 24 | [24-player-scaffold-bundle-loader.md](./issues/24-player-scaffold-bundle-loader.md) | Player scaffold and bundle loader | 4 |
| 25 | [25-gating-engine-store.md](./issues/25-gating-engine-store.md) | Gating engine and app state store | 4 |
| 26 | [26-progress-persistence.md](./issues/26-progress-persistence.md) | Progress persistence in localStorage | 4 |
| 27 | [27-player-i18n.md](./issues/27-player-i18n.md) | Player i18n runtime with ja/en catalogs | 4 |
| 28 | [28-layout-shell-navigation.md](./issues/28-layout-shell-navigation.md) | Layout shell, navigation, title screen, settings | 4 |
| 29 | [29-chapter-reader.md](./issues/29-chapter-reader.md) | Chapter reader view | 5 |
| 30 | [30-document-viewer.md](./issues/30-document-viewer.md) | Document evidence viewer | 5 |
| 31 | [31-image-viewer.md](./issues/31-image-viewer.md) | Image evidence viewer with zoom and pan | 5 |
| 32 | [32-sql-worker-service.md](./issues/32-sql-worker-service.md) | SQL worker service: sql.js isolation, timeout, caps | 5 |
| 33 | [33-sql-console-ui.md](./issues/33-sql-console-ui.md) | SQL console UI | 5 |
| 34 | [34-table-browser.md](./issues/34-table-browser.md) | No-code table browser | 5 |
| 35 | [35-interviews-ui.md](./issues/35-interviews-ui.md) | Character interviews UI | 5 |
| 36 | [36-question-judging-ui.md](./issues/36-question-judging-ui.md) | Question forms and client-side judging | 6 |
| 37 | [37-progression-notifications.md](./issues/37-progression-notifications.md) | Chapter progression and unlock notifications | 6 |
| 38 | [38-hint-system.md](./issues/38-hint-system.md) | Tiered hint system | 6 |
| 39 | [39-final-accusation-ending.md](./issues/39-final-accusation-ending.md) | Final accusation flow and ending screens | 6 |
| 40 | [40-sample-scenario-ja.md](./issues/40-sample-scenario-ja.md) | Sample scenario (Japanese): The Missing First Edition | 7 |
| 41 | [41-sample-scenario-en.md](./issues/41-sample-scenario-en.md) | Sample scenario (English adaptation) | 7 |
| 42 | [42-e2e-playthrough-suite.md](./issues/42-e2e-playthrough-suite.md) | End-to-end playthrough test suite | 7 |
| 43 | [43-security-verification.md](./issues/43-security-verification.md) | Security verification suite and SECURITY.md | 7 |
| 44 | [44-authoring-guide.md](./issues/44-authoring-guide.md) | Authoring guide | 7 |
| 45 | [45-deployment-guide-readme.md](./issues/45-deployment-guide-readme.md) | Deployment guide, README overhaul, live demo | 7 |
| 46 | [46-release-workflow.md](./issues/46-release-workflow.md) | npm release workflow with provenance | 7 |

\* Issue 22 has a cross-wave dependency on 24 (build assembles the prebuilt player dist). Waves are guidance; the dependency table below is authoritative.

## 3. Dependency table

`A ← B` means B depends on A. Transitive dependencies omitted.

| Issue | Depends on |
|---|---|
| 01 | — |
| 02 | 01 |
| 03 | 01, 02 |
| 04 | 01 |
| 05, 06, 07, 08 | 04 |
| 09 | 04, 05 |
| 10 | 04 |
| 11 | 04–10 |
| 12 | 01, 11 |
| 13 | 12, 18 |
| 14 | 07, 12, 18 |
| 15 | 05, 09, 12, 18 |
| 16 | 09, 10, 12, 13, 18 |
| 17 | 13, 14, 15, 16 |
| 18 | 11, 12 |
| 19 | 01 |
| 20 | 19, 21 |
| 21 | 17, 18, 19 |
| 22 | 17, 19, 24 |
| 23 | 19 |
| 24 | 01, 04, 11 |
| 25 | 05, 09, 24 |
| 26 | 25 |
| 27 | 24, 26 |
| 28 | 24, 25, 26, 27 |
| 29 | 25, 28 |
| 30 | 28 |
| 31 | 28 |
| 32 | 24 |
| 33 | 28, 32 |
| 34 | 28, 32, 33 |
| 35 | 25, 26, 28 |
| 36 | 10, 25, 26, 28 |
| 37 | 25, 28, 36 |
| 38 | 36 |
| 39 | 36, 37, 38 |
| 40 | 21, 22 |
| 41 | 40 |
| 42 | 22, 23, 29–35, 38, 39, 40, 41 |
| 43 | 13, 22, 42 |
| 44 | 18, 40, 42 |
| 45 | 22, 23, 42, 43 |
| 46 | 02, 03, 17, 22, 24, 43 |

Suggested parallel tracks after Wave 1: {12–18 compiler} ∥ {24–28 player foundation} ∥ {19, 23 CLI basics}; then {29–35 modalities} ∥ {21→20 CLI validate/init}; 22 lands once 17+24 exist.

## 4. Implementation waves

| Wave | Issues | Goal / demo at end of wave |
|---|---|---|
| 0 Foundation | 01–03 | CI-green empty monorepo with security rails |
| 1 Schema | 04–11 | `@datastory/schema` validates a fixture package in unit tests; JSON Schema autocompletes in VS Code |
| 2 Compiler | 12–18 | Fixture package compiles to bundle (story.json + case.db) with snapshot tests; every DS-code fixture fails correctly |
| 3 CLI | 19–23 | `init → validate → preview` works end-to-end from a terminal (build joins after 24) |
| 4 Player foundation | 24–28 | Player loads a hand-built bundle, shows shell + title screen, persists a trivial progress bit, switches ja/en |
| 5 Modalities | 29–35 | All five investigation modes usable against the fixture bundle |
| 6 Mechanics | 36–39 | Full game loop: answer → unlock → hint → accuse → ending |
| 7 Content & release | 40–46 | Both sample locales playable end-to-end; security verified; docs done; npm publish dry-run green |

## 5. Coverage: DESIGN.md sections → issues

| DESIGN.md section | Covered by issues |
|---|---|
| §4.1 Repository layout | 01 |
| §4.2/§4.3 Architecture & runtime | 12, 17, 24, 32 |
| §5.1–§5.2 Package layout, id rules | 04, 12 |
| §5.3 Manifest | 04 |
| §5.4 Chapters/unlock | 05, 15, 29 |
| §5.5 Evidence | 06, 12, 30, 31 |
| §5.6 Datapack | 07, 14, 34 |
| §5.7 Characters | 08, 35 |
| §5.8 Questions/hints | 09, 16, 36, 38, 39 |
| §6.1 Bundle layout | 17, 22 |
| §6.2 story.json | 17, 24 |
| §6.3 case.db | 14 |
| §7.1–§7.2 Gating runtime | 25, 37 |
| §7.3 Graph invariants | 15 |
| §7.4 Persistence | 26 |
| §8 Answer checking | 10, 16, 36 |
| §9.1 Shell | 28 |
| §9.2 Chapter reader | 29 |
| §9.3 Evidence browser | 30, 31 |
| §9.4 SQL console | 32, 33 |
| §9.5 Table browser | 34 |
| §9.6 Interviews | 35 |
| §9.7 Questions/hints/ending | 36, 38, 39 |
| §9.8 Settings/notifications/reset | 26, 28, 37 |
| §10.1 CLI global | 19 |
| §10.2 init | 20 |
| §10.3 validate | 21 |
| §10.4 build | 22 |
| §10.5 preview | 23 |
| §11 Diagnostics | 11, 18 |
| §12 i18n | 27 (+ catalog usage in every player issue) |
| §13.1 Threat model doc | 43 |
| §13.2 Sanitization | 13, 43 |
| §13.3 CSP | 22, 43 |
| §13.4 SQL robustness | 32, 34 |
| §13.5 Compiler hardening | 12, 14 |
| §13.6 Privacy | 26, 42, 43 |
| §13.7 Supply chain | 03, 46 |
| §13.8 Preview safety | 23 |
| §14 Testing strategy | 02, 42, 43 (+ per-issue Validation sections) |
| §15 Performance budgets | 02, 24, 33 |
| §16 Sample scenario | 40, 41 |
| §17 Release | 45, 46 |

## 6. Product-wide validation strategy

Layered per DESIGN §14; the load-bearing gates:

1. **Contract gates** (waves 1–2): schema accept/reject fixtures; compiler snapshot of the fixture bundle with pinned `--salt/--built-at`; one failing fixture per diagnostic code.
2. **Equivalence gate** (issues 10 + 42): the shared golden vectors pin normalization/hashing in Node unit tests (issue 10) and are re-verified inside real browsers by the E2E suite (issue 42) — protecting the judging contract that everything else stands on.
3. **Loop gate** (wave 6 exit): scripted fixture playthrough exercising every transition in DESIGN §7.2.
4. **Product gate** (issue 42): Playwright solves both sample locales on Chromium + WebKit, including the **no-SQL path** (DESIGN §16.3-1), reload persistence, reset, zero CSP violations, zero external requests.
5. **Security gate** (issue 43): sanitizer XSS corpus, CSP conformance, privacy assertions, SECURITY.md review.
6. **Release gate** (issue 46): publish dry-run, provenance, package `files:` audit.

## 7. Deferred v2 items

As fixed in DESIGN §3.3: investigation notebook/pinboard · LLM scenario-generation assistant · dialogue-triggered unlocks · audio/video/map evidence · teacher dashboard/analytics · `.dstory` archive distribution · authoring GUI · translation-sync tooling · sealed-epilogue cryptographic gating · URL deep links · progress export/import · additional locales · Japanese developer-docs translation.

## 8. Known unknowns that may create issues

From DESIGN §3.4: U1 sql.js/WebKit behavior on managed school devices · U2 localStorage quotas on managed Chromebooks · U3 Japanese name-matching tolerance · U4 print/PDF export demand · U5 non-root base-path hosting edge cases. Each is covered by a mitigation in existing issues; if a mitigation proves insufficient during implementation, open a new issue referencing the unknown's id (U1–U5) and this section.
