# Title

CSV to SQLite datapack builder

# Summary

Implement the compiler stage that turns `data/tables/*.csv` + `data/datapack.yaml` into a deterministic `case.db` SQLite file using sql.js in Node, with strict header matching, typed coercion, and row-level DS4xxx diagnostics.

# Context

DESIGN §6.3 requires byte-identical `case.db` for identical inputs (CI snapshots depend on it), and §13.5 requires streaming parsing with hard caps because CSVs are untrusted input. Using sql.js (not better-sqlite3) keeps the toolchain WASM-only — no native compilation for authors on any OS (consistent with ADR-003).

# Scope

- `packages/compiler/src/datapack.ts`: `buildCaseDb(datapack: Datapack, tablesDir: string): Promise<{db?: Uint8Array, diagnostics: Diagnostic[]}>` — `tablesDir` is the loader-validated absolute `data/tables/` directory (issue 12 already enforced realpath containment and the ≤ 5 MiB cap; this stage resolves `tables[].file` against `tablesDir` only and must not add any alternate file-reading path).
- Dep: `sql.js`, `csv-parse`. Unit tests + fixtures.

# Detailed Requirements

1. CSV parsing (`csv-parse` streaming API): RFC 4180, UTF-8, strip BOM, `relax_column_count: false`. Structural errors (unclosed quote, ragged row) → DS4001 with line number.
2. Header row must equal the declared column-name *set* (order-insensitive): missing column → DS4002, undeclared extra column → DS4005 (both list names).
3. Row caps: > 50,000 data rows → DS4006 and abort that table's parse (streaming — do not buffer the file first). Cell length > 4,000 chars → DS4004 (row+column in message).
4. Type coercion per declared type: `integer` — `/^-?\d+$/` and `Number.isSafeInteger(Number(v))` (else DS4003); `real` — finite `Number(…)` (else DS4003); `date` — `/^\d{4}-\d{2}-\d{2}(T\d{2}:\d{2})?$/` + real calendar date via `Date.UTC` round-trip (else DS4003; message shows expected formats); `text` — as-is. Empty string cell → SQL `NULL` for every type.
5. Table creation: `CREATE TABLE "name" ("col" TEXT|INTEGER|REAL, …)` in declared column order — identifiers always double-quoted via a shared `quoteIdent(name: SqlName)` helper exported from this module (DESIGN §5.6 quoting policy; grammar-validated names may still be SQL keywords); `PRIMARY KEY (…)` when declared (duplicate PK values → DS4007 with row number). Insert rows in CSV order using bound parameters, in one transaction per table.
6. Determinism (DESIGN §6.3): `PRAGMA page_size=4096` before first table; `PRAGMA user_version=1`; export via `db.export()`; **no** dates/times/random values written; test asserts two runs produce identical bytes.
7. Reject any SQL execution path that interpolates strings — table/column names come from schema-validated SqlName grammar (safe to inline), values always bound.

# Acceptance Criteria

- [ ] `valid-full` fixture produces a `case.db` loadable by sql.js with expected row counts per table, NULLs where cells were empty, and correct column affinities.
- [ ] One fixture per DS4001–DS4007 yields exactly that code with correct file/line.
- [ ] Byte-determinism test passes (two builds, `Buffer.compare === 0`).
- [ ] 50,001-row fixture aborts without materializing all rows in memory (heap assertion or read-counter spy).
- [ ] `students.csv` with reordered columns (vs declaration) still builds correctly in declared order.

# Validation

Fixture-driven unit tests; determinism test wired into CI. Cross-check one fixture DB with the `sqlite3` CLI manually during review (row count + schema) and note it in the PR.

# Dependencies

- 07, 12, 18

# Non-goals

- Player-side loading/querying of `case.db` (issues 32–34).
- CSV export back out of the tool (no such feature; avoids CSV-injection class entirely — note kept in SECURITY.md by issue 43).

# Design References

- DESIGN.md §5.6 (datapack rules), §6.3 (determinism), §13.5 (streaming caps), §11.1 (DS4xxx)
