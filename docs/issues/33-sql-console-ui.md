# Title

SQL console UI

# Summary

Implement the SQL console view per DESIGN §9.4.1: editor with run affordances, visible-tables schema sidebar, virtualized results table with truncation/timing feedback, error surface, session history, and database reset.

# Context

This is the SQL-education heart inherited from SQL Murder Mystery. The audience spans high-schoolers writing their first SELECT to teachers projecting results — feedback (errors, row counts, truncation) must be plain and non-intimidating, and rendering 5,000-row results must not jank (§15).

# Scope

- `packages/player/src/views/SqlConsole.tsx`, `packages/player/src/views/ResultsTable.tsx` (virtualized, shared with issue 34), `packages/player/src/views/SchemaSidebar.tsx` + CSS Modules; component tests alongside.

# Detailed Requirements

1. Editor: `<textarea>` (monospace, min 6 rows, auto-grow to 16) — no CodeMirror in v1 (budget §15); Run button + `Ctrl/Cmd+Enter`; running state disables Run and shows busy indicator tied to `SqlService.onStateChange`.
2. Schema sidebar: *visible* tables only (§9.4.2) — title, SQL name (copyable code style), `description`, column list `name (type)` with display titles as tooltips/secondary text; clicking a table name inserts it at the editor cursor.
3. Results: `ResultsTable` virtualized (windowing via fixed row height; no dependency or a micro-dependency ≤3 KB — prefer hand-rolled) showing NULL as a styled `NULL` badge; header = column names from the worker; footer status line: `sql.rowCount` / `sql.truncatedNotice` (with the limit) / `sql.elapsed` (`Intl.NumberFormat` ms).
4. Errors: message panel (`role="alert"`), preserves the statement; when `error.position` is present (rare — §9.4.3), render an inline caret line under the statement at that offset; timeout gets `sql.timeout` message with the configured seconds; `sql.restarted` explains the engine restarted.
5. History: session-only ring buffer (50). Recall rule (exact): `↑` recalls older entries only when the caret is at offset 0 with no selection; `↓` recalls newer only when the caret is at the end with no selection; the current unsent draft is preserved in slot 0 and restored when cycling back past the newest entry; recall never fires while composing (IME composition events guard). A history dropdown lists recent statements as a fallback affordance.
6. Reset DB: button with confirm dialog (`sql.resetConfirm`); on ack, clears results and shows an **inline** `sql.resetDone` status line (`role="status"`) — no toast dependency (issue 37 may migrate it later; this issue depends only on 28/32).
7. First-visit hint text (`sql.starter`): one-line nudge like "Try: SELECT * FROM entry_log" using the first visible table's real name; when no table is visible yet, hide the starter and show `sql.noVisibleTables` in the sidebar instead.
8. Strings in both catalogs; keyboard + `aria` labeling per §9.1 contract.

# Acceptance Criteria

- [ ] Query round-trip renders columns/rows; NULL badge visible; 5,000-row fixture renders ≤ 100 DOM rows (windowing assert) and completes initial render ≤ 1 s after the worker reply (DESIGN §15).
- [ ] Truncation and elapsed line render per fixture; error path keeps statement and announces via `role="alert"`.
- [ ] Sidebar lists only unlocked tables (fixture pre/post unlock); table-name insertion works at caret.
- [ ] `Ctrl+Enter` runs; history recall per the exact rule above (caret-position cases tested); 50-cap enforced; draft-slot restore works.
- [ ] Reset flow confirms, resets, re-queries pristine data, and announces the inline status (integration with issue 32 service stub).
- [ ] Error with `position` renders the caret line; without `position` renders message-only.
- [ ] axe smoke + ja/en strict-mode clean.

# Validation

Component tests with a stubbed `SqlService`; real end-to-end SQL path covered in issue 42.

# Dependencies

- 28, 32

# Non-goals

- Syntax highlighting/autocomplete (v2); saved queries; export of results (deliberately absent — no CSV export exists anywhere in the product, which removes the spreadsheet-formula-injection class; SECURITY.md records this, issue 43); multi-statement tabs.

# Design References

- DESIGN.md §9.4.1–§9.4.2 (console spec), §13.4 (limit UX), §15 (render budget)
