# Title

Datapack schema: tables, columns, display config

# Summary

Add the `data/datapack.yaml` schema to `@datastory/schema`: table definitions with SQL-safe names, typed columns, optional primary key, and the per-column display flags that drive the table browser.

# Context

DESIGN §5.6 defines the datapack as the bridge between authored CSVs and both query surfaces: the SQL console (issue 33) sees real SQLite tables; the table browser (issue 34) renders sort/filter controls from `columns[]` metadata. Name grammars keep identifiers simple and bounded, but may still be SQL keywords — all *generated* SQL therefore double-quotes identifiers via the shared `quoteIdent()` policy (DESIGN §5.6), implemented in issues 14/34.

# Scope

- `src/datapack.ts` in `packages/schema`, exports, unit tests.

# Detailed Requirements

1. `ColumnSchema` (`.strict()`): `name` SqlNameSchema · `type` enum `["text","integer","real","date"]` · `title` string 1..60 · `filterable` boolean default `true` · `sortable` boolean default `true`.
2. `TableSchema` (`.strict()`): `name` SqlNameSchema · `file` RelPathSchema ending `.csv` (case-insensitive) · `title` 1..80 · `description` ≤500 optional · `unlockedBy` IdSchema · `primaryKey` array of SqlName, 1..5 items, optional · `columns` array of ColumnSchema, 1..30 items.
   Refinements: column names unique within the table; every `primaryKey` entry must be a declared column name.
3. `DatapackFileSchema` (`.strict()`): `{ tables: TableSchema[] }` — 1..30 tables; refinements: table `name`s unique, table `file`s unique.
4. Compiled forms (`.strict()`): `CompiledTableSchema` `{name,title,description?,unlockedBy,columns:[{name,type,title,filterable,sortable}]}` (primaryKey is a DB-level concern, not shipped in story.json; §6.2) and `CompiledDatapackSchema` `{dbPath: "case.db", tables: CompiledTableSchema[]}` matching the full §6.2 datapack object.
5. Export inferred types (`Datapack`, `TableDef`, `ColumnDef`, `CompiledTable`, `CompiledDatapack`) via the barrel.

# Acceptance Criteria

- [ ] Accepts the DESIGN §5.6 example with defaults applied (`filterable`/`sortable` true when omitted).
- [ ] Rejects: table name `Entry_Log` (uppercase), `sqlite_seq` (reserved prefix), `file: "x.tsv"`, 31 columns, duplicate column names, `primaryKey: ["missing_col"]`, 31 tables, unknown keys, empty `columns`.
- [ ] `CompiledDatapackSchema.parse()` accepts the full §6.2 datapack object (including `dbPath`) and rejects unknown keys.

# Validation

Table-driven unit tests for every accept/reject case; type-level test that `ColumnDef["type"]` narrows to the four-value union.

# Dependencies

- 04

# Non-goals

- CSV parsing, header matching, row-level type coercion, row caps — issue 14 (DS4xxx).
- `unlockedBy` reference existence — issue 15.
- Table-browser UI behavior derived from these flags — issue 34.

# Design References

- DESIGN.md §5.6 (datapack), §6.2/§6.3 (compiled forms), §9.5 (display-config consumers)
