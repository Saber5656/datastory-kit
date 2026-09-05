# Title

Chapter progression and unlock notifications

# Summary

Implement the unlock feedback loop per DESIGN §9.8: toast notifications + `aria-live` announcements when correct answers release new content, nav badge synchronization, and the header progress indicator's live updates.

# Context

Gating is invisible machinery; this issue makes progress *felt*. When a learner's correct answer unlocks a chapter and three pieces of evidence, the game must say so immediately and point them there — this is the core reward beat of the play loop (§2.3 step 3).

# Scope

- `packages/player/src/notify/ToastHost.tsx`, `packages/player/src/notify/useUnlockToasts.ts` (+ `ToastHost.test.tsx`, `useUnlockToasts.test.tsx`), badge-clearing verification in `packages/player/src/components/Nav.tsx` (issue 28's component) + CSS Modules.

# Detailed Requirements

1. Toast host: bottom-right stack (top on narrow), max 3 visible (older collapse into a `+n` summary toast), auto-dismiss 6 s with pause-on-hover/focus, dismiss button; rendered inside the shell (issue 28) above main content; `prefers-reduced-motion` disables slide animations.
2. `useUnlockToasts` consumes the store's `pendingNotifications` queue via `consumeNotifications()` (issue 25). Batch shape is DESIGN §7.2's `NewlyUnlocked` (`{reason, chapters, documents, images, tables, characters}` of id arrays). Toast mapping per batch: `notify.newChapter {title}` (per chapter, title resolved from the bundle); one combined `notify.newEvidence {count}` where count = `documents.length + images.length`; `notify.newTable {count}`; `notify.newCharacter {count}`. **Toast text renders exclusively as React text nodes — never `SafeHtml`/`dangerouslySetInnerHTML`; a fixture title containing `<img onerror=…>` must appear escaped (DESIGN §13.1 boundary 3).**
3. Toast click targets (exact): chapter toast → `setActiveView({kind:"chapter", id: batch.chapters[0]})`; evidence toast → `{kind:"document", id: batch.documents[0]}` or, when only images unlocked, `{kind:"image", id: batch.images[0]}`; table toast → `{kind:"table", id: batch.tables[0]}`; character toast → `{kind:"character", id: batch.characters[0]}`. After navigation, focus moves to the target view heading (issue 28's contract).
4. `aria-live` announcement via issue 28's `useLiveAnnouncer()`: one combined sentence per batch (screen readers get one announcement, not four toasts).
5. Badge clearing (verification of issue 25/28 contracts): documents/images clear on `markEvidenceRead` (persisted, §7.1); chapters/tables/characters clear on `markSeen(kind, id)` fired by `setActiveView` — session-only `seen` sets in the store, never persisted (§7.4). Tests here assert both lifetimes (reload fixture keeps evidence cleared, resets table badge).
6. Header progress `n/N` chapters and the Questions section count update reactively (already derived — verify subscription).
7. Solved transition: batches with `reason: "finalAccusation"` are **not** toasted (engine contract, §7.2) — issue 39 owns the full-screen moment; the live region is also skipped for that batch.
7. Strings in both catalogs.

# Acceptance Criteria

- [ ] Simulated correct answer unlocking {1 chapter, 2 docs, 1 table} produces the mapped toasts (evidence count = 2), one combined live-region sentence, and the exact `setActiveView` calls above (store assertions).
- [ ] 4+ simultaneous toasts collapse to 3 + summary.
- [ ] XSS fixture: a chapter title `<img src=x onerror=alert(1)>` renders escaped in the toast; no `dangerouslySetInnerHTML` in `src/notify/**` (grep assert).
- [ ] Badges clear per the defined rules (evidence persisted, others session-only — reload fixture keeps evidence cleared, resets table badge).
- [ ] `reason: "finalAccusation"` batch produces zero toasts and zero live-region output.
- [ ] Reduced-motion honored (class/style assertion); axe smoke + ja/en strict-mode clean.

# Validation

Component tests per above; full-loop feel review during issue 42's manual pass.

# Dependencies

- 25, 28, 36

# Non-goals

- Ending/celebration screens (39); achievement systems (never in v1 scope); sound effects (v2 candidate, off by default if ever).

# Design References

- DESIGN.md §9.8 (notifications), §7.2 (newlyUnlocked), §9.1 (live region placement)
