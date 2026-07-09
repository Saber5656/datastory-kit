# Research: embedded SQL engine options for the browser (as of 2026-07)

Purpose: fact-check for [ADR-003](../decisions/ADR-003-sqljs-engine.md). The product needs an in-browser SQL engine for read-mostly, classroom-scale datasets (§5.6 caps: ≤30 tables, ≤50k rows/table), loaded fresh per session from a compiled `case.db`, with **no persistence requirement** (progress lives in `localStorage`, not the database — DESIGN §7.4).

## Findings (verified 2026-07-10)

| Option | Status / size | Fit notes |
|---|---|---|
| **sql.js** | v1.14.0, MIT, ~13.5k stars; "the standard solution for running SQL in the browser"; wasm+js ≈ 1–1.5 MB raw. Maintenance slow-moving. | In-memory only — exactly matches our copy-per-session + reset-from-bytes architecture (§9.4.3). Proven dual use in Node (compiler, issue 14) and browser (player, issue 32). |
| **Official sqlite3 WASM** (sqlite.org/wasm) | Officially supported member of the SQLite deliverables family; OO (`oo1`) API similar to sql.js. Its `opfs` VFS needs COOP/COEP headers; `opfs-sahpool` avoids them but is single-connection. | Persistence VFS features are irrelevant to us (in-memory VFS needs no special headers). **Designated fallback engine** if sql.js maintenance stalls; swap surface is contained behind the §9.4.3 worker protocol. |
| **wa-sqlite** | Recommended by current ecosystem write-ups for *persistent* browser SQLite (OPFSCoopSyncVFS; OPFSWriteAheadVFS 2026-04, Chrome-only concurrent reads). | Optimizes the axis we don't use (persistence/concurrency). |
| **DuckDB-WASM** | `duckdb-eh.wasm` ≈ 18 MB raw (a ~6.4 MB variant exists; ≈ 1.8–2.8 MB gzipped over the wire). Analytical (OLAP) engine; best performance paths want COOP/COEP. | 5–10× sql.js payload; analytical strengths unneeded at classroom scale; SQLite dialect is also the SQL-education mainstream. |
| **PGlite** | Postgres-in-WASM (~3 MB); younger project. | Postgres dialect diverges from school SQL-education norms; revisit for v2 analytics scenarios. |

## Decision impact

- ADR-003 keeps **sql.js** as v1 engine; payload claims corrected to the verified numbers (the earlier "30+ MB" DuckDB figure was an over-claim — realistic raw wasm is ~6–18 MB).
- ADR-003 gains an explicit fallback clause: official sqlite3 WASM build, swap contained to `packages/player/src/sql/worker.ts` + issue 14's builder.
- Re-check sql.js release activity at the v1 release gate (issue 46 runbook precondition is *not* added — this is an ADR-level watch item, not a release blocker).

## Sources

- [sql.js](https://sql.js.org/) · [sql-js/sql.js (GitHub)](https://github.com/sql-js/sql.js/)
- [sqlite3 WebAssembly & JavaScript Documentation Index](https://sqlite.org/wasm)
- [SQLite Wasm in the browser backed by OPFS (Chrome for Developers)](https://developer.chrome.com/blog/sqlite-wasm-in-the-browser-backed-by-the-origin-private-file-system)
- [The Current State Of SQLite Persistence On The Web: May 2026 Update (PowerSync)](https://powersync.com/blog/sqlite-persistence-on-the-web)
- [DuckDB WASM large file size (observablehq/framework#1260)](https://github.com/observablehq/framework/issues/1260) · [duckdb/duckdb-wasm](https://github.com/duckdb/duckdb-wasm)
