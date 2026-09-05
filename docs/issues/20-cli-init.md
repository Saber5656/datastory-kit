# Title

datastory init command and starter templates

# Summary

Implement `datastory init <dir>`: scaffold a new scenario package from the `minimal` or `library-case` template in the requested locale, with heavily commented files that teach the format inline.

# Context

`init` is the author's first touch (DESIGN §2.2); template quality decides whether the format feels approachable. Templates must always validate cleanly — a broken first experience is fatal for adoption — so their tests run them through the real `validate` pipeline (issue 21).

# Scope

- `src/commands/init.ts` in `packages/cli`; template content under `packages/cli/templates/{minimal,library-case}/{ja,en}/`.

# Detailed Requirements

1. Signature: `datastory init <dir> [--locale ja|en] [--template minimal|library-case] [--force]` — defaults `ja`, `minimal` (DESIGN §10.2).
2. Behavior: create `<dir>` if missing. If it exists and is non-empty: refuse with DS1101 (exit 1) unless `--force`; with `--force`, write only files that do not already exist, warn (stderr) for each skip. Never overwrite.
3. Templates are literal file trees (no templating engine); the only substitution is the scenario `id`, derived from the directory basename by this exact algorithm: NFKC-normalize → lowercase → replace every run of characters outside `[a-z0-9]` with a single `-` → trim leading/trailing `-` → truncate to 64 chars → if the result fails `idPattern` (empty or 1 char), use `my-scenario` and warn (stderr, human) with the original name quoted.
4. `minimal` template (per locale): `scenario.yaml` with all **required** fields active (`formatVersion`, `id`, `locale`, `title`, `version`) and every **optional** field shown as a commented-out example with an explanatory comment; 1 chapter (`unlock: start`); 1 text gate question + final accusation; `questions.yaml` comments explaining hints/accepted/spoilerGuard; the full DESIGN §5.1 directory tree (`evidence/docs/`, `evidence/images/`, `data/tables/`, `characters/portraits/`) present with `.gitkeep` files; plus a `README-first.md` at the package root pointing to the authoring guide (legal: the loader ignores unknown root files, §5.1). Comment language matches `--locale`.
5. `library-case` template (per locale): a trimmed one-chapter excerpt of the sample scenario (issue 40's content, reduced: 1 chapter, 2 documents, 1 image, 1 table, 1 character, 1 gate question + final) demonstrating every file kind in realistic use. The template image obeys the real constraints (PNG/JPEG/WebP, no SVG, ≤ 2 MiB) and passes the real loader pipeline. Until issue 40 lands, seed with placeholder-but-valid content marked `TODO(sample-sync)`; issue 40's **scope** includes syncing this template.
6. Post-scaffold output (stderr, suppressed by `--quiet` except warnings): file list + next-steps block (`datastory validate <dir>` → `datastory build <dir>` → `datastory preview <dir>/dist`).
7. Templates ship inside the published package (`files` includes `templates/`).

# Acceptance Criteria

- [ ] `init x --locale ja` then `datastory validate x` exits 0 with zero warnings — for all four template×locale combinations (automated test).
- [ ] Non-empty dir without `--force` exits 1 with DS1101; with `--force` existing files untouched (mtime/content assert) and missing ones created.
- [ ] Directory named `My Case!` yields id `my-case` (algorithm above); directory named `!` falls back to `my-scenario` with the warning.
- [ ] Comments in ja templates are Japanese; en templates English.
- [ ] Scaffolded tree matches DESIGN §5.1 layout exactly.

# Validation

Automated template-validate round-trip tests (the load-bearing check); manual read-through of template comments by a reviewer for tone/clarity.

# Dependencies

- 19, 21 (validate used in tests; land 21 first)

# Non-goals

- Interactive prompts/wizard (v2); template gallery beyond the two (v2).

# Design References

- DESIGN.md §10.2 (init), §2.2 (author journey), §5 (format being taught)
