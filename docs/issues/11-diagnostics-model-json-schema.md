# Title

Diagnostic model and JSON Schema export

# Summary

Define the shared `Diagnostic` type in `@datastory/schema` and add the build step that exports editor-consumable JSON Schema files for every authored YAML kind (manifest, images, datapack, character, questions).

# Context

Every compiler stage reports problems as `Diagnostic` values rendered by the reporters (issue 18) in the exact §11.2/§11.3 formats. Separately, ADR-005 promises authors editor autocomplete: generating JSON Schema from the Zod definitions keeps that promise without a second source of truth.

# Scope

- `packages/schema/src/diagnostic.ts`, `packages/schema/scripts/export-json-schema.ts`, generated `packages/schema/schemas/*.schema.json` (committed), drift check test. Covers the five authored `.yaml` file kinds only — chapter/document *frontmatter* schemas are a non-goal (YAML editors cannot validate frontmatter blocks without extra tooling).

# Detailed Requirements

1. `Diagnostic` type + Zod schema: `{ code: string (/^DS\d{4}$/), severity: "error"|"warning", file: string, line?: number (int ≥1), column?: number (int ≥1), message: string, hint?: string }`. Serialization contract note (consumed by issue 18): the `--json` reporter emits absent `line`/`column`/`hint` as `null`, never as omitted keys (DESIGN §11.3).
2. Helpers: `isError(d)`, `sortDiagnostics(ds)` (by file path asc, then line asc nulls-first, then code) — the canonical ordering used by both reporters (§11.2).
3. JSON Schema export script (dev dependency `zod-to-json-schema`): emits to `packages/schema/schemas/`:
   - `scenario.schema.json` (manifest), `images.schema.json`, `datapack.schema.json`, `character.schema.json`, `questions.schema.json`.
   - Each with `$id: "https://datastory-kit.dev/schemas/v1/<name>.schema.json"` and `title` set.
4. Generated files are **committed**; a Vitest test regenerates in-memory and diffs against the committed files (drift ⇒ failing test with "run pnpm --filter @datastory/schema generate:schemas" message). Scripts: `packages/schema/package.json` defines `generate:schemas`; the root `package.json` adds `generate:schemas` delegating via `pnpm --filter @datastory/schema generate:schemas`.
5. `schemas/` included in the published package `files`.
6. README section in `packages/schema/README.md`: 5-line snippet showing VS Code `yaml.schemas` mapping (`scenario.yaml`, `evidence/images.yaml`, `data/datapack.yaml`, `characters/*.yaml`, `questions.yaml`).

# Acceptance Criteria

- [ ] All five schema files generate deterministically (two consecutive runs byte-identical) and are committed.
- [ ] Drift test fails when a Zod schema changes without regeneration (verified once by scratch edit).
- [ ] In VS Code with the YAML extension and the documented mapping, typing an unknown key in `scenario.yaml` shows a squiggle (manual check, screenshot in PR).
- [ ] `sortDiagnostics` ordering matches §11.2 (test with shuffled fixtures).

# Validation

`pnpm --filter @datastory/schema test` (unit + drift tests) and `pnpm --filter @datastory/schema generate:schemas` twice with a byte-diff check for determinism. Manual VS Code check evidenced by a screenshot committed to the PR description (not the repo).

# Dependencies

- 04, 05, 06, 07, 08, 09, 10 (all schemas exist to export)

# Non-goals

- The DS-code catalog with messages/hints and the pretty/JSON reporters — issue 18.
- JSON Schemas for chapter/document Markdown frontmatter (see Scope note).
- Publishing schemas to a URL (the `$id`s are identifiers, not fetched; hosting is a v2 nicety).

# Design References

- DESIGN.md §11.1–§11.3 (diagnostics), ADR-005 (editor support promise)
