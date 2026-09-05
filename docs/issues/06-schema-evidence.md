# Title

Evidence schemas: documents and images

# Summary

Add schemas for document-evidence frontmatter (`evidence/docs/*.md`) and the image manifest (`evidence/images.yaml`) to `@datastory/schema`, plus their compiled `story.json` counterparts.

# Context

DESIGN §5.5 defines the two static-evidence kinds. Documents carry prose (rendered by issue 13); images are binary assets whose *metadata* is authored in `images.yaml` (binary validation — magic bytes, size — is the compiler loader's job, issue 12). Alt text is mandatory for accessibility.

# Scope

- `src/evidence.ts` in `packages/schema`, exports, unit tests.

# Detailed Requirements

1. `DocumentFrontmatterSchema` (`.strict()`): `id` IdSchema · `title` 1..120 · `category` string 1..40 · `unlockedBy` IdSchema · `date` string ≤40 optional · `source` string ≤80 optional.
2. `ImagesFileSchema` (`.strict()`): `{ images: ImageEntry[] }`, max 40 entries (DESIGN §5.5 cap), unique `id`s and unique `file`s enforced via refinement.
3. `ImageEntrySchema` (`.strict()`): `id` IdSchema · `file` RelPathSchema, additionally must end in `.png|.jpg|.jpeg|.webp` (case-insensitive) · `title` 1..120 · `alt` string 1..500 **required** · `caption` string ≤500 optional · `unlockedBy` IdSchema.
4. Compiled forms (both `.strict()`): `CompiledDocumentSchema` `{id,title,category,unlockedBy,date?,source?,bodyHtml}` and `CompiledImageSchema` `{id,title,alt,caption?,unlockedBy,src}`. `src` validation: must start with `media/`, satisfy RelPathSchema (no `..`, no absolute, no `\`), end in `.png|.jpg|.jpeg|.webp` (case-insensitive), and must not contain `:` (rejects `https:`/`data:` URLs).
5. Export all inferred types via the barrel.

# Acceptance Criteria

- [ ] Accepts the DESIGN §5.5 examples.
- [ ] Rejects: missing `alt`, `file: "../x.png"`, `file: "/abs.png"`, `file: "x.svg"`, `file: "x.gif"`, duplicate image ids, duplicate files, 41 images, unknown keys.
- [ ] Extension check is case-insensitive (`X.JPG` accepted).
- [ ] `CompiledDocumentSchema.parse()` / `CompiledImageSchema.parse()` accept the exact §6.2 docs/images fragments; reject unknown keys, `src: "/media/x.jpg"`, `src: "../x.jpg"`, `src: "https://x/y.jpg"`, `src: "data:image/png;base64,…"`.

# Validation

Table-driven unit tests covering every accept/reject case above.

# Dependencies

- 04

# Non-goals

- Binary validation (magic bytes, ≤2 MiB, extension/content match) — issue 12 (DS6xxx).
- `unlockedBy` reference existence — issue 15.
- Rendering document bodies — issue 13.

# Design References

- DESIGN.md §5.5 (evidence), §6.2 (compiled), §13.5 (image hardening, for the boundary note)
