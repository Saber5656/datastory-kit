# Title

CLI scaffold: argument parsing, exit codes, output conventions

# Summary

Create `packages/cli` with the `datastory` binary skeleton: command registration, global flags, the §10.1 exit-code contract, stdout/stderr discipline, and the process-level error handler. Individual commands land in issues 20–23.

# Context

DESIGN §10.1 fixes CLI-wide behavior so all four commands feel identical and machine callers (CI, editor tasks, agents) can rely on exit codes and stream separation. Getting this scaffolding right once prevents four divergent implementations.

# Scope

- Package scaffold `packages/cli` (deps of this issue: `commander`, `picocolors` only — `@datastory/compiler`/`@datastory/schema` wiring arrives with issues 21/22, keeping this issue's dependency at 01).
- `packages/cli/src/index.ts` (bin entry), `packages/cli/src/context.ts` (output helpers), `packages/cli/src/exit.ts`; test helper `packages/cli/test/helpers/run-cli.ts`; tests via spawned processes.

# Detailed Requirements

1. Package: `"bin": {"datastory": "dist/index.js"}` with shebang `#!/usr/bin/env node`; `"engines": {"node": ">=20"}`; ESM.
2. Commander program: name `datastory`, version from package.json (`--version`/`-v`), help via both `--help` and `-h` at global and per-command level (help text to stderr per §10.1); four subcommands registered as stubs that print "not implemented" and exit 3 until their issues land.
3. Global flags on every command: `--quiet` (suppress non-error stderr output), `--no-color` (also honor `NO_COLOR` env and non-TTY auto-detect; commands pass the resolved color decision into the issue 18 reporters and use `picocolors` elsewhere).
4. Stream discipline (§10.1): human/progress/diagnostic output → **stderr**; machine payloads (`--json` now, `--porcelain` values if any command adds them later) → **stdout** only. Provide `ctx.human()`, `ctx.machine()` helpers; ESLint rule bans raw `console.*` in `packages/cli/src`.
5. Exit codes via a single `exitWith(code: 0|1|2|3)`: 0 success · 1 content errors · 2 usage error (commander errors mapped here: unknown command/flag, bad flag value) · 3 internal error. Uncaught exceptions/rejections → handler prints a short message + "Please report: <repo issues URL>" + stack when `DEBUG=1`, exits 3. Test hook: when `DATASTORY_TEST_THROW=1`, a hidden no-op command path throws deliberately so spawn tests can exercise the handler.
6. Windows/POSIX path handling: never print `\`-joined paths in machine output (normalize to `/`).
7. Test harness helper in `packages/cli/test/helpers/run-cli.ts`: `runCli(args: string[], opts?: {cwd?: string, env?: Record<string,string>, timeoutMs?: number}): Promise<{code: number, stdout: Buffer, stderr: Buffer}>` (spawns the built binary, byte-accurate capture) — shared by issues 20–23 tests.

# Acceptance Criteria

- [ ] `datastory --version` prints the package version to stdout, exit 0.
- [ ] `datastory nosuchcmd` exits 2 with usage on stderr; stdout empty.
- [ ] `datastory validate` (stub) exits 3 with "not implemented".
- [ ] With `DATASTORY_TEST_THROW=1`, the handler path exits 3 and prints the report pointer; with `DEBUG=1` added, the stack appears.
- [ ] `--quiet` suppresses human output but never machine output; `NO_COLOR=1` output contains no ANSI escapes (assert on bytes).
- [ ] ESLint `no-console` guard active for the package.

# Validation

Spawn-based tests per above on Node 20/22; manual `--help` review for wording.

# Dependencies

- 01

# Non-goals

- Command implementations (20–23); network features (none exist by design, §10).

# Design References

- DESIGN.md §10.1 (global behavior, exit codes, streams)
