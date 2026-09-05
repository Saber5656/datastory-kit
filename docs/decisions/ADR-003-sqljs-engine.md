# ADR-003: sql.js (SQLite WASM) as the embedded query engine

- Status: Accepted
- Date: 2026-07-10
- Deciders: Design agent (conservative default; user delegated runtime detail decisions)

## Context

Two v1 investigation modes ("SQL console" and "table browser") query tabular case data inside the browser (ADR-001: no server). Candidate embedded engines: sql.js (SQLite compiled to WASM), DuckDB-WASM, PGlite (Postgres WASM). Datasets are small (classroom scale: thousands of rows, a handful of tables). School devices may be low-powered; school networks slow, so payload size matters.

## Decision

Use **sql.js** as the only query engine in v1.

1. The compiler builds a standard **SQLite database file** (`case.db`) from authored CSVs (deterministically — see DESIGN.md §6.3).
2. The player loads `case.db` into sql.js running inside a **dedicated Web Worker** (never on the main thread).
3. The worker enforces robustness limits (execution timeout with worker termination + respawn, result row cap, statement length cap — DESIGN.md §13.4).
4. Both the SQL console and the table browser query the same worker; the table browser generates parameterized SQL internally.
5. The learner's database instance is an in-memory copy; a "reset database" action reloads the pristine `case.db`.

## Consequences

Positive:

- ~1–1.5 MB engine payload (~0.5 MB gzipped) vs DuckDB-WASM's ~6–18 MB wasm (~2–3 MB gzipped transfer) — see [research/embedded-sql-engine-2026.md](../research/embedded-sql-engine-2026.md); smaller and simpler on school networks, cacheable offline.
- sql.js's known limitation — in-memory only, no persistence — is a **non-issue by architecture**: the design loads a pristine `case.db` copy per session and resets from cached bytes (DESIGN.md §9.4.3); learner progress never lives in the database.
- SQLite dialect is the lineage of SQL Murder Mystery and the most commonly taught dialect; abundant learner-facing documentation.
- `case.db` is a plain SQLite file: authors can inspect it with any SQLite tool; the same library builds it in Node (no native compilation).
- No COOP/COEP header requirements (needed by threaded DuckDB-WASM and the official build's `opfs` VFS), keeping GitHub Pages hosting trivial.

Negative:

- No analytical SQL extensions (window functions exist in modern SQLite, but no advanced OLAP). Acceptable for classroom-scale mysteries.
- sql.js API is synchronous; isolation and timeouts must come from the worker boundary, which the design mandates.
- Whole-database in memory; fine at the enforced dataset size caps (DESIGN.md §5.6).
- sql.js maintenance is slow-moving (v1.14.0 as of mid-2026). Mitigation: the DESIGN.md §9.4.3 worker protocol fully encapsulates the engine — the designated fallback is the **official `sqlite3` WASM build** (sqlite.org/wasm, officially supported; its in-memory VFS needs no special headers); a swap touches only the worker module and the compiler's db builder.

## Alternatives considered

- **Official sqlite3 WASM build**: officially maintained and the strongest challenger; chosen as fallback rather than primary because sql.js has longer dual Node+browser precedent for exactly this in-memory pattern and a smaller integration surface today. Revisit if sql.js stalls (watch item in the research note).
- **wa-sqlite**: best-in-class for *persistent* browser SQLite (OPFS VFS work); persistence is exactly what this product does not need.
- **DuckDB-WASM**: superior analytical SQL, but ~6–18 MB wasm payloads, higher memory, and COOP/COEP complications for best performance — mis-sized for classroom datasets.
- **PGlite**: promising, but Postgres dialect diverges from the SQL-education mainstream for schools, and the project is younger; revisit for v2 if analytics-grade scenarios appear.
- **In-JS query engine (alasql etc.)**: not real-SQL fidelity; teaching-value loss.
