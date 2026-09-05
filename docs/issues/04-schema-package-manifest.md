# Title

@datastory/schema scaffold and scenario manifest schema

# Summary

Create the `packages/schema` package (Zod-based, zero I/O) with the shared id grammars and the `scenario.yaml` manifest schema, including the `settings` caps. This package is the single source of type truth for compiler and player.

# Context

ADR-002/ADR-005 make `@datastory/schema` the contract layer: Zod schemas validate authored objects (compiler side) and the inferred TS types drive the player. DESIGN §5.2 fixes identifier grammars; §5.3 fixes the manifest shape. Later schema issues (05–09) add one file-kind each; this issue establishes the package conventions they follow.

# Scope

- Package scaffold: `packages/schema/package.json`, `tsconfig.json`, `vitest.config.ts`, build setup.
- `src/ids.ts`, `src/manifest.ts`, `src/index.ts` barrel, unit tests.

# Detailed Requirements

1. Package: name `@datastory/schema`, `"type": "module"`, exports ESM + types (use `tsc` build to `dist/`, `exports` map with `types` condition). Runtime dependency: `zod` only. `"sideEffects": false`, `"files": ["dist"]`, `"license": "MIT"`.
2. `src/ids.ts`:
   - `idPattern = /^[a-z0-9][a-z0-9-]{1,63}$/` exported as regex + `IdSchema` (Zod string with the pattern; exact wording of error messages is not part of the contract).
   - `sqlNamePattern = /^[a-z][a-z0-9_]{0,29}$/` + `SqlNameSchema`; reject names starting with `sqlite_` via refinement.
   - `SemverSchema`: `/^\d+\.\d+\.\d+$/`.
   - `RelPathSchema`: non-empty string; must not start with `/`; must not match `/^[A-Za-z]:/` (Windows drive-letter absolutes); must not contain `\`, any `..` segment, or NUL; max 200 chars. (Filesystem realpath checks are the compiler's job — this is shape-level only.)
3. `src/manifest.ts` — `ScenarioManifestSchema` per DESIGN §5.3, exact fields:
   - `formatVersion`: literal `1`.
   - `id`: IdSchema. `locale`: enum `["ja","en"]`. `title`: string 1..120. `version`: SemverSchema.
   - `description`: string 1..1000 optional. `authors`: array of string 1..100, max 10, optional. `estimatedPlayMinutes`: int 1..600 optional. `targetAudience`: string ≤60 optional. `contentWarnings`: array of string ≤200, max 10, default `[]`. (All caps are canonical per DESIGN §5.3 field-caps paragraph.)
   - `settings.allowExternalLinks`: boolean default `false`.
   - `settings.sql.rowLimit`: int 1..5000 default 5000; `settings.sql.timeoutMs`: int 100..10000 default 5000. `settings` and `settings.sql` fully optional with defaults applied via `.default({})` so parsing always yields concrete values.
   - `.strict()` on every object (unknown keys are errors — authors get typo protection).
4. Barrel `src/index.ts` re-exports schemas + inferred types (`ScenarioManifest`, etc.).
5. Package convention doc comment (top of `index.ts`): "No I/O, no Node/browser-specific APIs (except WebCrypto via globalThis in the answer module, issue 10)."

# Acceptance Criteria

- [ ] `ScenarioManifestSchema.parse()` accepts the DESIGN §5.3 example (converted to JS object).
- [ ] A *minimal* manifest (only `formatVersion`, `id`, `locale`, `title`, `version`) parses and yields exactly `contentWarnings: []`, `settings.allowExternalLinks: false`, `settings.sql.rowLimit: 5000`, `settings.sql.timeoutMs: 5000` (asserted deep-equal — this is the defaults-application proof).
- [ ] Rejects (each with a test): bad id casing, `formatVersion: 2`, unknown top-level key, `locale: "fr"`, `version: "1.0"`, `rowLimit: 10001`, 121-char title.
- [ ] `RelPathSchema` rejects `C:/x.png` and `C:\x.png`; accepts `images/x.png`.
- [ ] `pnpm --filter @datastory/schema build`, `… test`, `… lint`, `… typecheck` each exit 0; package builds valid ESM + `.d.ts`.
- [ ] No runtime dependency other than `zod`.

# Validation

Unit tests as above (table-driven accept/reject). Import smoke test from a scratch Node script consuming the built `dist/`.

# Dependencies

- 01

# Non-goals

- Other file-kind schemas (05–09), normalizer (10), JSON Schema export (11).
- YAML parsing (compiler, issue 12) — this package validates already-parsed objects.

# Design References

- DESIGN.md §5.2 (id/reference rules), §5.3 (manifest)
- ADR-002 (package layout), ADR-005 (format principles)
