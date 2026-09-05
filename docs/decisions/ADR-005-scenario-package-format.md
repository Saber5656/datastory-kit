# ADR-005: Declarative scenario package (YAML + Markdown + CSV) compiled to a static bundle

- Status: Accepted
- Date: 2026-07-10
- Deciders: Product owner (declarative hand-written authoring, 2026-07-10 requirements interview); concrete format design by design agent

## Context

The user decided v1 authoring is hand-written declarative content (no LLM dependency in the engine; authors — human or their own AI agents — write files). The format must be: writable in a text editor, diffable in git, validatable with precise errors, and mechanically compilable. Story text needs rich-text (headings, emphasis, ruby for furigana); case data is tabular; structure (chapters, gates, dialogue, questions) is hierarchical.

## Decision

1. **Authoring format = a directory** ("scenario package") with a fixed layout (DESIGN.md §5.1): `scenario.yaml` manifest, `chapters/*.md`, `evidence/docs/*.md`, `evidence/images/*` + `evidence/images.yaml`, `data/tables/*.csv` + `data/datapack.yaml`, `characters/*.yaml`, `questions.yaml`.
2. **YAML** (YAML 1.2 core schema, safe parsing, no custom tags/anchors across documents) for structure; **Markdown** (CommonMark + GFM tables + `<ruby>` support) for prose; **CSV** (RFC 4180, UTF-8) for data.
3. **Compilation** (`datastory build`) produces a static bundle: `story.json` (all structure + prose pre-rendered to sanitized HTML), `case.db` (SQLite), copied image assets, and the prebuilt player (DESIGN.md §6).
4. **Plaintext answers never reach the bundle**: `questions.yaml` holds accepted answers in plaintext in the *source* package; the compiler stores only salted hashes in `story.json` (ADR-007).
5. **IDs are explicit**: every chapter/evidence/character/topic/question declares a kebab-case `id`; all cross-references are by id and are validated at compile time (unknown-ref = build error).
6. The schema package exports **JSON Schema** files generated from the Zod definitions so editors (VS Code YAML extension) can autocomplete and validate while authoring.

## Consequences

Positive:

- Authors need no toolchain knowledge beyond files + one CLI; packages are git-friendly and reviewable.
- Deterministic, LLM-free compilation makes CI snapshot testing of bundles possible.
- Explicit ids + compile-time reference validation catch the most common authoring mistakes before learners ever see them.
- The format is exactly the interface a future v2 LLM generator must target — schema-first design keeps that door open.

Negative:

- Multi-file authoring has more surface than one big file; mitigated by `datastory init` templates and `validate` diagnostics with file/line positions.
- Pre-rendering Markdown at compile time means content fixes require a rebuild (acceptable: build is seconds).

## Alternatives considered

- **Single JSON/YAML mega-file**: hostile to prose authoring and merge conflicts.
- **TS/JS scenario-as-code**: rejected — executing author code breaks the security model (untrusted scenario packages must not execute during compile or play).
- **Zip archive format (`.dstory`)**: deferred to v2 as a distribution convenience; the directory remains the source of truth.
