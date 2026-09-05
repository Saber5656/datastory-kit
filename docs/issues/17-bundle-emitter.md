# Title

Bundle emitter: story.json, media, case.db

# Summary

Implement the final compiler stage: assemble the validated, rendered, hashed scenario into the on-disk bundle — `story.json` per DESIGN §6.2, content-hash-renamed media files, `case.db` — with the integrity block and reproducibility overrides.

# Context

The emitter owns the compiler↔player contract (§6). The player's bundle loader (issue 24) validates `story.json` against the compiled schemas from issues 05–09, so shape drift fails fast on both sides. CLI `build` (issue 22) wraps this stage and adds the player dist; the emitter itself stays UI-agnostic.

# Scope

- `src/emit.ts` in `packages/compiler`: `emitBundle(compiled, outDir, opts: {builtAt?: string}): Promise<{report: BuildReport, diagnostics: Diagnostic[]}>`.
- `compileScenario(rootDir, opts)` orchestrator composing stages 12→13→14→15→16→17.
- Unit + integration tests.

# Detailed Requirements

1. Media pipeline: images/portraits deduplicated by content — group by `sha256(bytes)`; the emitted filename is `media/<id>-<first 6 hex of that sha256><ext>` where `<id>` is the id of the **first authored reference** in stable traversal order (images in `images.yaml` order, then characters in filename order); all references to identical bytes point at that one file.
2. `story.json` assembly exactly per §6.2: chapters sorted by `order`; evidence docs sorted by (category, title); images/tables/characters/questions in authored file order; `sql` block from manifest settings (defaults applied); `salt` from issue 16; `integrity.builtAt` = `opts.builtAt ?? new Date().toISOString()`.
3. `integrity.contentHash` — exact algorithm: take every **source** file of the scenario package the loader consumed (never outputs); sort by relative path (POSIX `/` separators, UTF-8 byte order); for each, feed `utf8(relPath) + 0x00 + uint64-LE(byteLength) + bytes` into one running SHA-256; hex-encode. Salt and builtAt are never part of the hash (§6.2 reproducibility note).
4. Serialization: `JSON.stringify(story, null, 2)` + trailing newline (stable key order via explicit object construction — no key-order surprises).
5. `case.db` written iff the datapack has ≥ 1 table (§6.1 "present iff tables exist"); `datapack` omitted from story.json otherwise (player treats SQL/table views as absent).
6. Output safety (non-skippable here even though the CLI adds its own guards, §10.4): resolve `outDir` via `realpath` after creation; refuse (throw usage error) when it resolves inside the scenario package root — except the package's own `dist/` default (§10.4 exception); every write path must stay under the resolved `outDir` (assert per write).
7. `BuildReport`: `{outDir, files: [{path, bytes}], totalBytes, contentHash, warnings: Diagnostic[]}` — `files[].path` relative to `outDir`, POSIX separators, sorted; consumed by CLI `--json` (issue 22).
8. Size warning DS1104 when totalBytes > 100 MiB.
9. Orchestrator `compileScenario(rootDir, opts: {validateOnly?: boolean, salt?: string, builtAt?: string, outDir?: string}): Promise<{report?: BuildReport, diagnostics: Diagnostic[]}>` composes stages 12→13→14→15→16→(17): any `error`-severity diagnostic from any stage aborts before `emitBundle`; `validateOnly: true` runs every diagnostic-producing stage but writes nothing and skips media copying. This is the single public compile API — the CLI (issues 21/22) calls only this.

# Acceptance Criteria

- [ ] `valid-full` fixture with pinned `--salt`/`--built-at` emits a byte-stable bundle across two runs (story.json, case.db, media names all identical) — committed snapshot test.
- [ ] `valid-minimal` (no data/images) emits no `case.db`, no `media/`, and story.json omits `datapack.dbPath`.
- [ ] story.json of `valid-full` parses against the compiled Zod schemas (issues 05–09) with zero issues.
- [ ] Grep-level test: no plaintext accepted answers from the fixture appear anywhere under `outDir`.
- [ ] Two images with identical bytes produce one media file referenced twice.
- [ ] Emitting into the package root is refused.

# Validation

Integration snapshot tests per above, wired into CI (this snapshot is the wave 2 exit gate from ISSUE_PLAN §4).

# Dependencies

- 13, 14, 15, 16

# Non-goals

- Copying player dist / CSP injection / base-path rewriting (issue 22).
- `.dstory` archive output (v2).

# Design References

- DESIGN.md §6.1–§6.3 (bundle format), §10.4 (build flags contract), §11.1 (DS1104)
