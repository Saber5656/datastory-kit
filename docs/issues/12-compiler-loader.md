# Title

Compiler scaffold and hardened scenario-package loader

# Summary

Create `packages/compiler` and its first stage: load a scenario package directory into validated in-memory objects — with the full input-hardening posture of DESIGN §13.5 (path containment, YAML safety, size caps, image sniffing) and DS1xxx/DS2xxx/DS6xxx diagnostics.

# Context

The loader is trust boundary 1 (DESIGN §13.1): a scenario package is untrusted input running against the *author's* machine. Every later compiler stage assumes the loader has already produced schema-valid objects and safe file references, so this issue carries most of the compiler's security weight.

# Scope

- Package scaffold `packages/compiler` (Node-only, ESM, deps: `@datastory/schema`, `yaml`, `gray-matter` or equivalent frontmatter extraction, `csv-parse` reserved for issue 14).
- `src/loader.ts` + `src/types.ts` (`LoadedScenario` aggregate), unit tests with fixture packages under `packages/compiler/test/fixtures/`.

# Detailed Requirements

1. Entry: `loadScenario(rootDir: string): Promise<{scenario?: LoadedScenario, diagnostics: Diagnostic[]}>` — never throws for content problems; throws only for programmer errors.
2. Layout discovery per DESIGN §5.1 — the loader discovers and parses exactly these inputs: required `scenario.yaml`, `chapters/*.md` (≥1), `questions.yaml`; optional `evidence/docs/*.md`, `evidence/images.yaml` (required iff `evidence/images/` non-empty), `evidence/images/*` (referenced via `images[].file`), `data/datapack.yaml` (required iff `data/tables/` non-empty), `data/tables/*` (referenced via `tables[].file`), `characters/*.yaml`, `characters/portraits/*` (referenced via `portrait`). Anything else — including `dist/` — is ignored without diagnostics (§5.1 loader rules). Missing required file → DS1001 (one per file, expected path in message); zero chapter files → DS1001 with file `chapters/` and message "expected at least one chapters/*.md".
3. **Path containment** (DS1002): every file the loader touches must satisfy `realpath(file).startsWith(realpath(rootDir) + sep)`; symlinks escaping the root are rejected. Author-declared relative paths (`images[].file`, `tables[].file`, `portrait`) are resolved against their fixed base dirs only.
4. **YAML safety** (DS1003): parse with the `yaml` package, core schema, `uniqueKeys: true`, alias/merge expansion capped (`maxAliasCount: 100`), and **custom tags rejected** (any non-core tag → DS1003, per DESIGN §13.5); parse errors → DS1004 with line/column from the parser.
5. **Size caps** checked via `stat` *before* reading (DESIGN §5.5/§5.6/§13.5, all canonical there): CSV ≤ 5 MiB (DS1005), image ≤ 2 MiB (DS6002), any YAML/MD ≤ 1 MiB (DS1006), total package ≤ 200 MiB (DS1007).
6. **Frontmatter extraction** for `chapters/*.md` and `evidence/docs/*.md`: YAML frontmatter block parsed under the same YAML safety; body kept as raw Markdown string for issue 13. Missing/empty frontmatter → DS1008.
7. **Schema validation**: run the issue 04–09 schemas on every parsed object; Zod issues mapped to DS2001 diagnostics carrying file path + YAML line when obtainable (map Zod paths to yaml CST nodes via the `yaml` package's line counter; line `null` acceptable where impractical).
8. **Image validation** (DS6001/DS6003): magic-byte sniff per DESIGN §13.5 table; extension/content mismatch → DS6003; portraits validated identically.
9. **Package-level caps**: ≤ 20 character files (DS1009), ≤ 40 images enforced at schema level already, ≤ 30 tables at schema level.
10. `LoadedScenario` shape: `{rootDir, manifest, chapters: {front, bodyMd, file}[], documents: {front, bodyMd, file}[], images: {entry, absPath}[], datapack?: {def, tablesDir}, characters: {def, portraitAbsPath?}[], questions: QuestionsFile (parsed+validated object), questionsFilePath: string, mediaFiles: Map<string, string>}` — `mediaFiles` maps *image-evidence id → absolute path* and *character id → portrait absolute path*; CSVs are not in the map (issue 14 consumes `datapack.tablesDir` + `def.tables[].file` directly). Downstream stages receive parsed objects, never re-read author files.
11. Determinism: directory listings sorted by filename before processing so diagnostic order is stable.

# Acceptance Criteria

- [ ] Golden fixture `valid-minimal/` (1 chapter, 1 final accusation, no data/images/characters) loads with zero diagnostics.
- [ ] Golden fixture `valid-full/` (every feature incl. images, datapack, characters) loads with zero diagnostics.
- [ ] One fixture per diagnostic listed above (DS1001–DS1009, DS2001, DS6001–DS6003) produces **exactly** that code at the expected file, asserted by tests.
- [ ] Symlink-escape fixture (symlink to a file outside the package) is rejected with DS1002 on POSIX; test skipped on Windows runners with a comment.
- [ ] A 6 MiB CSV fixture fails fast (stat check) without reading the file body (assert via timing-free spy on read calls).
- [ ] YAML fixture with a custom tag (`!!python/object` style) → DS1003.
- [ ] Static-safety greps pass over `packages/compiler/src`: no `eval(`/`new Function(`, no `child_process` import, no dynamic `require`/`import()` of author-derived paths (three explicit grep-level tests per DESIGN §13.5).

# Validation

Fixture-driven unit tests per above; run under Node 20 and 22 in CI.

# Dependencies

- 01, 11 (Diagnostic type; schemas 04–09 arrive via 11's dependency chain)

# Non-goals

- Markdown rendering (13), CSV parsing/coercion (14), cross-reference/graph checks (15), hashing (16), emission (17), reporters (18).

# Design References

- DESIGN.md §5.1–§5.7 (layout), §13.5 (hardening), §11.1 (code ranges), §13.1 boundary 1
