# Title

Diagnostics catalog and pretty/JSON reporters

# Summary

Create the checked-in DS-code catalog (code → message template, severity, remedy hint) and the two output renderers: human-readable pretty format and the stable `--json` contract of DESIGN §11.

# Context

Every compiler stage (12–17) emits `Diagnostic` values; this issue gives them one registration point and one presentation layer, so codes stay unique, documented, and greppable. The authoring guide (issue 44) generates its error-reference table from this catalog — single source of truth.

# Scope

- `src/catalog.ts`, `src/report-pretty.ts`, `src/report-json.ts` in `packages/compiler`; unit tests.

# Detailed Requirements

1. Catalog: `const CATALOG: Record<DsCode, CatalogEntry>` with entries for every code referenced by issues 12–17 and 20–22: the **required list** is DS1001–DS1009, DS1101, DS1102, DS1104, DS2001, DS3001–DS3010, DS3101, DS4001–DS4007, DS5001, DS5101, DS5102, DS5103, DS6001–DS6003, DS7002, DS7003, DS7101 — a test asserts exactly this set is present (not grep-derived, so codes owned by not-yet-written CLI code are still covered). Codes follow §11.1 ranges; `x0xx`=error, `x1xx`=warning — a unit test enforces the convention and severity/catalog agreement.
2. Exported API shapes (exact):
   - `type DsCode = \`DS${number}\`` narrowed by the catalog keys; `type CatalogEntry = {severity: "error"|"warning", template: string, hint: string}` (hint always present);
   - `type DiagnosticLocation = {file: string, line?: number, column?: number}`;
   - `diag(code: DsCode, params: Record<string, string|number>, loc: DiagnosticLocation): Diagnostic` — interpolates `{param}` placeholders; unknown code or missing param throws (programmer error);
   - `reportPretty(ds: Diagnostic[], opts: {color: boolean}): string` and `reportJson(ds: Diagnostic[]): string`.
3. Pretty reporter (§11.2): `error DS4003  data/tables/entry_log.csv:17  message` + indented `hint:` line; ordering via `sortDiagnostics` (issue 11); ANSI colors (red error / yellow warning) enabled only via `opts.color` (callers derive from `isTTY`/`NO_COLOR`); final summary line `N errors, M warnings`. **Author-controlled strings (file paths, params, messages built from package content) are neutralized before output: C0/C1 control characters and ESC are replaced with `␛`-style visible escapes** — a hostile package must not be able to inject terminal escape sequences (DESIGN §13.1 boundary 1); tests cover a filename and a CSV value containing `\x1b[31m`.
4. JSON reporter (§11.3): exact envelope `{version: 1, diagnostics: [...], summary: {errors, warnings}}`; absent `line`/`column`/`hint` serialize as `null` (issue 11 contract); output produced only by `JSON.stringify` (inherently escape-safe); stable field order; no ANSI ever.
5. Doc generation: `scripts/catalog-to-md.ts` emits a Markdown table (code, severity, message, hint) → committed at `docs/reference/diagnostics.md` with a drift test (same pattern as issue 11).
6. Later issues add codes by extending the catalog; the uniqueness test guards collisions.

# Acceptance Criteria

- [ ] Required-list test passes (exact code set above); additionally a grep-driven test asserts every `DS\d{4}` literal in `packages/compiler/src` (and `packages/cli/src` when it exists) is catalog-registered.
- [ ] Pretty output matches the §11.2 example byte-for-byte for a fixed fixture (snapshot, `color: false`).
- [ ] ANSI-injection fixtures (filename and message param containing `\x1b[31m`) render with visible escapes, no raw ESC byte in output (byte-level assert).
- [ ] JSON output validates against a Zod schema of the §11.3 envelope with `null` serialization; snapshot committed.
- [ ] Severity/range convention test passes; duplicate-code registration fails a test.
- [ ] `docs/reference/diagnostics.md` generated, committed, drift-tested.

# Validation

Unit + snapshot tests including a color-forced snapshot (`opts.color: true`, ANSI codes asserted in the snapshot itself — objective, no eyeballing); one manual terminal screenshot in the PR is informational only.

# Dependencies

- 11, 12 (12 and 18 land together in practice; 13–17 build on both)

# Non-goals

- CLI wiring of `--json` flags (issues 21/22); localized diagnostic messages (English-only in v1 — authors are also served by the ja authoring examples in issue 44).

# Design References

- DESIGN.md §11.1–§11.3 (catalog, formats)
