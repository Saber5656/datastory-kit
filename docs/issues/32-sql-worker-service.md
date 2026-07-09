# Title

SQL worker service: sql.js isolation, timeout, caps

# Summary

Implement the Web Worker that owns the sql.js database plus its main-thread supervisor client, per the DESIGN §9.4.3 protocol and §13.4 robustness rules: init from `case.db`, exec with row cap, reset, timeout-kill-respawn.

# Context

Both query surfaces (SQL console 33, table browser 34) share this service. Isolation in a worker keeps the UI responsive during heavy queries and makes the timeout enforceable (`worker.terminate()` is the only reliable way to stop synchronous WASM execution). This is robustness engineering, not a security boundary (§13.4) — the learner only affects their own in-memory copy.

# Scope

- `packages/player/src/sql/worker.ts` (worker side), `packages/player/src/sql/SqlService.ts` (main-thread client), shared types in `packages/player/src/sql/protocol.ts`; unit + real-worker integration tests (see Validation).

# Detailed Requirements

1. Protocol exactly per §9.4.3: requests `init` (carries `dbBytes: ArrayBuffer`), `exec` (learner-typed single statement), `query` (generated SQL + `params: (string|number|null)[]`, bound execution — used by the table browser, issue 34), `reset`; responses per §9.4.3 with `Cell = null | number | string` (BLOB → literal string `"[BLOB]"`). No `schema` op — sidebar metadata comes from `story.json` (§9.4.1).
2. Worker execution model (§9.4.3): `initSqlJs` with `locateFile` → relative `sql-wasm.wasm` (bundle root, §6.1). Statements run via `db.prepare()` + `stmt.step()` loop collecting rows up to `rowLimit` then stopping with `truncated: true` (never materialize-then-truncate; this is what makes the cap effective). `exec`: if `prepare` leaves trailing SQL, return an error ("one statement at a time"). `query`: `stmt.bind(params)` before stepping — values only, identifiers are caller-side (§5.6 quoting). Mutating statements step to completion and ack with zero columns. `elapsedMs` via `performance.now()`. SQL errors → `{ok:false, error:{message, position?}}` (`position` included only if sql.js provides it; usually absent).
3. Supervisor (`SqlService`): caches the pristine `dbBytes` on `init` (main-thread copy — worker memory dies with `terminate()`); promise-per-request map keyed by id; enforces §13.4:
   - statement length > 10,000 chars → reject client-side without messaging the worker;
   - per-request timer = bundle `sql.timeoutMs`; on expiry: `terminate()`, reject the in-flight promise, respawn worker, auto re-`init` from the cached bytes, reject queued requests;
   - serialize requests (one in flight; queue the rest) — timeout attribution stays unambiguous.
   Rejections use `class SqlServiceError extends Error { code: "sql.tooLong" | "sql.timeout" | "sql.restarted" | "sql.error"; }` (codes double as chrome keys; worker-reported SQL errors surface as `sql.error` with the worker message attached).
4. `reset` restores pristine bytes (drop learner mutations) and acks; supervisor exposes `onStateChange` (`ready|busy|restarting|failed`) for UI status.
5. `rowLimit`/`timeoutMs` come from `story.json` `sql` block (§6.2), already cap-clamped by the compiler; the supervisor re-clamps defensively (≤5000 rows, ≤10000 ms).
6. No datapack in bundle → service not constructed; views 33/34 absent (§6.2 note).

# Acceptance Criteria

- [ ] Real-worker integration tests (Vitest browser mode or Playwright component runner — a genuine `Worker`, not a shim) cover at minimum: init fixture db → `exec("SELECT …")` returns expected columns/rows; `truncated` true at rowLimit+1; bad SQL error surfaces; multi-statement `exec` rejected; `query` with bound params returns the filtered rows; timeout (`WITH RECURSIVE` infinite query) → restart → next query succeeds (`busy→restarting→ready` observed); `reset` after `DROP TABLE` restores queryability. Cross-browser coverage may defer to issue 42, but these real-worker paths may not.
- [ ] 10,001-char statement rejected without worker round-trip (spy), error `code === "sql.tooLong"`.
- [ ] Queued request behind a timeout rejects with `code === "sql.restarted"`.
- [ ] BLOB cell arrives as the literal string `"[BLOB]"`.
- [ ] Protocol types shared by both sides (single module import; no `any` at the boundary).

# Validation

Real-worker integration tests as above wired into CI; supervisor pure-logic tests (timers, queueing) in plain Vitest with a mock worker.

# Dependencies

- 24

# Non-goals

- Any UI (33/34); SQL statement allow/deny-listing (§9.4.2: full SQL permitted by design); multi-database support.

# Design References

- DESIGN.md §9.4.3 (protocol), §13.4 (limits), §6.1/§6.2 (wasm + sql config), ADR-003
