# Title

Chapter reader view

# Summary

Implement the Story view per DESIGN §9.2: render the selected unlocked chapter's sanitized body, list chapters in `order`, show a locked placeholder for locked chapters, and link to that chapter's question status.

# Context

Chapters carry the narrative spine and are the only gated navigation nodes visible while locked (§9.1). The reader must handle Japanese long-form text well (ruby furigana from issue 13, comfortable measure) since the primary audience is school learners.

# Scope

- `packages/player/src/views/ChapterReader.tsx` + `ChapterReader.module.css` + `packages/player/src/views/ChapterReader.test.tsx`.

# Detailed Requirements

1. Chapter list (within the Story nav section, provided by issue 28) ordered by `order`; the reader view renders the active chapter.
2. Unlocked chapter: `<h2>` title, body rendered **only** through the shared `SafeHtml` component (no new `dangerouslySetInnerHTML` call sites — DESIGN §13.2), then a "Questions in this chapter" block listing **all** questions where `question.chapter === activeChapter.id` (not only gate questions): status icon (✓ answered / ○ open) + a text snippet derived from `promptHtml` by tag-stripping text extraction, truncated to 60 Unicode code points (the snippet is rendered as plain text, never as HTML), linking via `setActiveView({kind:"questions", id: question.id})` (issue 25's action; issue 36's view handles the anchor).
3. Locked chapter (lock row clicked): placeholder with lock icon and `chapter.lockedExplain` chrome string including the count of unanswered questions in its `afterQuestions` list (never their text — §9.2), plus a button `setActiveView({kind:"questions"})` (unfiltered — filtering is not a defined Questions-view capability).
4. Reading typography: max line length ~38em, `line-height ≥ 1.8` for ja, ruby renders with adequate `ruby-position`; print-friendly is out of scope.
5. Reading a chapter marks nothing (chapters have no read-tracking in v1; only evidence does — §7.1).
6. Epilogue (`unlock: solved`) chapters render with a subtle "Epilogue" tag (chrome key) once unlocked.
7. All strings via i18n; keys added to both catalogs.

# Acceptance Criteria

- [ ] Fixture: chapters render in `order` regardless of array order; active chapter body shows ruby markup intact.
- [ ] Question status block reflects engine state and navigates to the Questions view with the right anchor (store assertion).
- [ ] Locked chapter shows count-only placeholder — prompt text of locking questions never appears in DOM (explicit negative assertion).
- [ ] Epilogue tag appears only when solved.
- [ ] axe smoke: no serious/critical violations.
- [ ] Bilingual render without missing keys (test mode).

# Validation

`pnpm --filter @datastory/player test -- ChapterReader` with the `valid-full` fixture bundle; then `pnpm --filter @datastory/player dev` with the ja fixture and visually check: locked row shows no title, unlocked chapter shows ruby rendering, line-height/measure comfortable for ja prose, question snippets are plain text.

# Dependencies

- 25, 28

# Non-goals

- Question answering itself (36); unlock notifications (37); chapter-complete celebrations (39 handles the ending only).

# Design References

- DESIGN.md §9.2 (reader), §9.1 (locked visibility rules), §7.1 (chapterComplete)
