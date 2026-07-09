# Title

No-code table browser

# Summary

Implement the click-to-investigate view per DESIGN §9.5: pick a visible table, then sort, filter, and search it through generated **parameterized** SQL against the shared worker — the primary investigation surface for learners who don't write SQL.

# Context

The user chose school education as the primary audience; this view is what makes gate questions solvable for SQL-less learners (the "no modality lock-out" invariant, §16.3-1). Filter affordances derive mechanically from the datapack column config (§5.6) so authors control the UI without writing UI.

# Scope

- `packages/player/src/views/TableBrowser.tsx`, `packages/player/src/views/FilterRow.tsx`, `packages/player/src/views/useTableQuery.ts` (SQL generation hook) + CSS Modules; reuses `ResultsTable` from issue 33 (hard dependency — dependency table updated); unit + component tests alongside.

# Detailed Requirements

1. Table picker: visible tables as tabs/select (title + row count fetched once via `SELECT COUNT(*)`).
2. Grid: paged, 50 rows/page (§9.5), pager with page `n/N`; column headers show `title` with SQL `name` as secondary text.
3. Sorting exactly per DESIGN §9.5 (single algorithm, no client-side re-sort): `sortable` columns toggle asc/desc/none, implemented as SQL `ORDER BY` — `integer`/`real` numeric, `date` lexicographic ISO, `text` with `COLLATE NOCASE`; the ja code-point-order limitation is documented in a code comment referencing §9.5.
4. Filters per column config (§9.5): `filterable` text column → distinct-value dropdown when `SELECT COUNT(DISTINCT "col")` ≤ 50 else "contains" input; `integer/real` → min/max inputs; `date` → from/to date inputs. Filter row visually attached under headers; active filters get a clear-all chip.
5. Global search: single input; `WHERE ("t1" LIKE ? ESCAPE '\' OR "t2" LIKE ? …)` across text columns; user input escaped for `%`/`_`/`\`.
6. `useTableQuery`: composes `SELECT "cols" FROM "table" WHERE … ORDER BY … LIMIT 50 OFFSET ?` and executes via issue 32's `op:"query"` (bound execution — the protocol op already exists). Two hard rules (§13.4/§5.6): every user-influenced **value** travels as a bound parameter, never concatenated; every **identifier** comes only from `story.datapack.tables[].name`/`columns[].name` (grammar-validated) and is double-quoted via the shared `quoteIdent()` helper — bound params cover values, quoting covers identifiers.
7. Empty result: `table.noMatch` state with a "clear filters" action and a nudge button (`table.hintNudge`) that navigates via `setActiveView({kind:"questions"})` — navigation intent only; hint UI itself is issue 38 and is not a dependency.
8. Debounce text inputs 300 ms; all queries route through the shared serialized service (issue 32) — UI shows the busy state.
9. Strings in both catalogs; full keyboard operability (filters, sort buttons with `aria-sort`).

# Acceptance Criteria

- [ ] Unit tests on `useTableQuery`: generated SQL + params for each filter type, combination, pagination, sort states; `%`/`_` escaping verified.
- [ ] Distinct-≤50 dropdown vs contains-input switch behaves per fixture data.
- [ ] Component flow: filter to an expected row set, sort by date desc, paginate — against the real fixture db through a worker stub honoring bound params.
- [ ] Empty-state + clear-all restore full grid.
- [ ] `aria-sort` reflects state; axe smoke + bilingual clean.
- [ ] No string-interpolated user input reaches SQL (grep + code review checklist item).

# Validation

`pnpm --filter @datastory/player test -- useTableQuery TableBrowser` (hook + component tests); grep check `grep -rn "FROM \${" packages/player/src/views` returns nothing (no template-interpolated SQL); axe + strict-i18n checks as in sibling issues. The no-SQL solve path is exercised end-to-end in issue 42.

# Dependencies

- 28, 32, 33

# Non-goals

- Cross-table joins in the UI (learners use the SQL console for that); saved filters; CSV export (absent by design).

# Design References

- DESIGN.md §9.5 (table browser), §5.6 (column config), §13.4 (parameterization), §16.3-1 (no lock-out)
