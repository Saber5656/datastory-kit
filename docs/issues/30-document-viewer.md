# Title

Document evidence viewer

# Summary

Implement the document half of the Evidence view per DESIGN §9.3: a category-grouped list of visible documents with unread indicators, and a reader pane with title, metadata line, and sanitized body.

# Context

Documents are the baseline investigation mode (interview transcripts, memos, articles). The "new/unread" affordance drives the investigative loop — learners must notice when gates release new material (with issue 37's toasts pointing here).

# Scope

- `packages/player/src/views/DocumentList.tsx`, `packages/player/src/views/DocumentReader.tsx` + CSS Modules; component tests alongside.

# Detailed Requirements

1. List: group `visibleDocuments()` by `category` and order titles within each group per DESIGN §9.3 — since the emitter already sorts docs by (category, title) (§6.2), preserving bundle order implements this; group order = first appearance in bundle order. Each row: title + unread dot when id ∉ `readEvidence`.
2. View contract: `activeView {kind:"document"}` without `id` renders the list; with `id` renders the reader; an `id` that is unknown or not currently visible falls back to the list. Opening a document fires `markEvidenceRead(id)` (engine action; idempotent).
3. Reader: `<h2>` title; metadata line composed of `date` and `source` when present (chrome-labelled, e.g. `evidence.date`/`evidence.source`); body rendered only through the shared `SafeHtml` (no re-sanitizing, no new `dangerouslySetInnerHTML` — DESIGN §13.2); back-to-list affordance on narrow layouts.
4. Empty state (no visible documents yet): `evidence.emptyDocs` chrome message hinting that more evidence appears as the story progresses.
5. Long documents: no artificial truncation or nested scrolling; the main panel scrolls.
6. Keyboard: list is a proper list of buttons/links; reader heading receives focus on open (§9.1 contract).
7. All strings via i18n; keys in both catalogs.

# Acceptance Criteria

- [ ] Fixture: only documents of unlocked chapters appear; simulated unlock adds the new group/row.
- [ ] Unread dot clears after opening (facts assert + rerender check).
- [ ] Metadata line renders for docs with date/source and is absent otherwise (no dangling separators).
- [ ] Empty state shows for a fresh minimal fixture; unknown/hidden `id` in `activeView` falls back to the list.
- [ ] axe smoke passes; ja and en render with zero missing-key throws in the strict i18n test mode (keys added to both catalogs).

# Validation

Component tests per above; manual read-through of a long ruby-heavy fixture doc in the dev server.

# Dependencies

- 28

# Non-goals

- Image evidence (31); full-text search across evidence (v2); annotations/pinboard (v2 notebook).

# Design References

- DESIGN.md §9.3 (evidence browser), §7.1 (readEvidence), §6.2 (docs ordering)
