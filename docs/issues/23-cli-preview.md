# Title

datastory preview static server

# Summary

Implement `datastory preview <dir>`: a loopback-by-default static file server for built bundles with correct MIME types (including `application/wasm`), traversal-safe path resolution, and no caching — the author's play-test loop. Non-loopback binding is possible but explicitly warned (classroom LAN use case).

# Context

Browsers won't run WASM/fetch from `file://`, so authors need a one-command server (DESIGN §2.2 step 5, §3.4 U1). It must follow §13.8: this is a dev tool that should still be safe-by-default because teachers will run it on school machines.

# Scope

- `src/commands/preview.ts` in `packages/cli` using `node:http` (no server framework dependency).

# Detailed Requirements

1. Signature: `datastory preview <dir> [--port <n>] [--host <addr>] [--open] [--strict-port]` — defaults port 4173, host `127.0.0.1` (DESIGN §10.5).
2. Startup: verify `<dir>/index.html` and `<dir>/story.json` exist (else exit 2 with "not a datastory build — run datastory build first"). All human output (URL, port, warnings) goes to **stderr** via `ctx.human()` (issue 19); stdout stays empty; `--quiet` suppresses everything except warnings and errors. `--open` launches the default browser best-effort: the URL is constructed only from the validated host/port (never user strings), invoked via platform-specific `spawn` (`open`/`xdg-open`/`start`) with the URL as a single argument — no shell interpolation; failure is a warning.
3. Port busy: auto-increment up to +10 unless `--strict-port` (then exit 1). Print the final port (stderr).
4. Request handling (§13.8):
   - Decode URI inside try/catch — malformed percent-encoding → 404, never a crash; reject `%00`; normalize `\` and `%5c` to rejection before containment checks; resolve against root; `path.relative(root, resolved)` must not start with `..` — else 404 (never 403, no oracle).
   - Serve regular files only: `lstat` first; symlinks are followed only if their `realpath` stays under the served root **and** targets a regular file — otherwise 404; directory request → serve `index.html` only for `/`; no directory listings; anything else 404.
   - MIME map: html, js, css, json, wasm (`application/wasm`), png, jpg/jpeg, webp, db (`application/octet-stream`), map, txt, ico. Unknown → `application/octet-stream`.
   - Headers on every response: `Cache-Control: no-store`, `X-Content-Type-Options: nosniff`.
5. `--host 0.0.0.0` (or any non-loopback): print a prominent warning that the site is reachable from the LAN.
6. Graceful shutdown on SIGINT/SIGTERM; port released (test polls).
7. HEAD supported; other methods → 405.

# Acceptance Criteria

- [ ] Serves a stub bundle: `/`, `/story.json`, `/sql-wasm.wasm` (content-type asserted), `/media/x.jpg` all 200 with `no-store` + `nosniff`.
- [ ] Traversal attempts 404: `/../secret`, `/%2e%2e/secret`, `/a/../../secret`, encoded backslash variants; symlink-outside-root fixture 404s.
- [ ] Non-build directory → exit 2 with the guidance message.
- [ ] Busy-port auto-increment works; `--strict-port` exits 1.
- [ ] POST → 405; `/nope` → 404; no directory listing at `/media/`.
- [ ] Non-loopback host prints the warning.
- [ ] `HEAD /story.json` returns the same headers as GET with an empty body; `POST /` → 405, and both carry `no-store` + `nosniff`.
- [ ] SIGINT terminates the process and releases the port (test polls until reconnectable).
- [ ] `--open` test: spawn call mocked/spied — invoked with exactly the printed URL; simulated failure degrades to a warning, server keeps running.

# Validation

HTTP-level integration tests (node fetch against a spawned server) covering the matrix above.

# Dependencies

- 19

# Non-goals

- HTTPS, HTTP/2, compression, live-reload (v2 niceties); serving the scenario *source* dir (only built bundles).

# Design References

- DESIGN.md §10.5 (preview), §13.8 (server safety), §3.4 U1 (wasm MIME unknown)
