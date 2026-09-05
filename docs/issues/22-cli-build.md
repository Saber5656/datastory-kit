# Title

datastory build command with CSP injection

# Summary

Implement `datastory build <dir>`: validate, compile, emit the bundle (issue 17), assemble the prebuilt player dist, and inject base path + CSP meta + `<html lang>` into `index.html` — producing the final deployable static site.

# Context

This command joins the two halves of the product (DESIGN §4.2): compiler output + `@datastory/player` dist. It also owns two safety-relevant edits: the CSP meta tag (§13.3) and the destructive-output guard (§10.4) that keeps `-o ~/Documents` from ending badly.

# Scope

- `src/commands/build.ts` in `packages/cli`; player-dist resolution helper; spawn-based tests using a stub player dist fixture until issue 24 provides the real one.

# Detailed Requirements

1. Signature: `datastory build <dir> [-o|--out <dir>] [--base-path <path>] [--salt <32hex>] [--built-at <iso8601>] [--json]` — defaults per DESIGN §10.4 (`<dir>/dist`, `./`). Malformed `--salt` (fails `SALT_PATTERN`) or `--built-at` (not ISO 8601) → usage error, exit 2, before any compile work.
2. Sequence: call issue 17's `compileScenario(dir, {salt, builtAt, outDir})` — it validates internally and aborts with diagnostics before emitting on any error (CLI maps that to exit 1, printing via the issue 18 reporters). On success: copy player dist into `outDir` → transform `index.html`.
3. Player dist resolution: `require.resolve("@datastory/player/package.json")` → its `dist/`; fail exit 3 with a clear message if absent (broken install). `playerVersion` in the report is read from that same package.json.
4. `index.html` transforms (string-safe, parse5 or regex-on-known-markers — prefer parse5):
   - Inject `<meta http-equiv="Content-Security-Policy" content="default-src 'none'; script-src 'self' 'wasm-unsafe-eval'; style-src 'self'; img-src 'self'; connect-src 'self'; font-src 'self'; base-uri 'none'; form-action 'none'; object-src 'none'">` as the **first** child of `<head>` (§13.3, exact string).
   - Set `<html lang="<scenario locale>">`.
   - Rewrite asset URL prefixes with `--base-path` (trailing-slash normalized); default `./` keeps everything relative (§6.1).
   - Set `<title>` to the scenario title (HTML-escaped).
5. Output guard (§10.4, realpath-based per §13.5): resolve both the scenario package root and `--out` with `realpath` (after creating `--out` if missing); `--out` inside the package is refused with exit 2 **except** the exact default `<dir>/dist` (the sole permitted inside-package location — the loader ignores it, §5.1). If `--out` exists and is non-empty: wipe it only when it contains a `story.json` (prior build marker); otherwise refuse with DS1102 (exit 1). All writes are asserted to stay under the resolved `--out`.
6. `--json` build report to stdout, schema `BuildCommandReportSchema` exported from `packages/cli/src/commands/build.ts`: issue 17's `BuildReport` fields + `{playerVersion: string, cspInjected: true}`; stdout carries only this JSON.
7. Idempotence: rebuilding into the same out dir succeeds (marker present → wipe → rebuild).

# Acceptance Criteria

- [ ] Build of `valid-full` + stub player dist yields §6.1 layout exactly; `index.html` first head child is the CSP meta (byte compare of the tag), `lang` set, title escaped.
- [ ] `--base-path /repo/` rewrites asset prefixes; default build contains no absolute paths (grep test).
- [ ] Out-dir guard: `-o <dir>/evidence` (inside package) → exit 2; default `<dir>/dist` → allowed; existing non-build dir → exit 1 + DS1102; existing prior build → wiped and rebuilt; `--salt zz` → exit 2 before any compile output.
- [ ] Pinned `--salt/--built-at` builds are byte-identical across two runs (excluding nothing — full-tree hash compare).
- [ ] Validation errors abort before any write to `--out`.
- [ ] `--json` report matches its Zod schema snapshot.

# Validation

Spawn-based tests with the stub dist; issue 42 later validates real-player builds end-to-end. Manual: open a built bundle via `preview` and check DevTools shows zero CSP violations at title screen (recorded in PR).

# Dependencies

- 17, 19, 24. (Unit tests may exercise the assembly against a minimal fixture dist for speed, but the issue is complete only when `datastory build` assembles the real `@datastory/player` dist — issue 24 is a hard dependency.)

# Non-goals

- Deployment (issue 45 docs); watch/incremental builds (v2); service-worker offline packaging (v2 — plain static files already work offline-after-load).

# Design References

- DESIGN.md §10.4 (build), §13.3 (CSP), §6.1–§6.2 (bundle), §3.4 U5 (base-path unknown)
