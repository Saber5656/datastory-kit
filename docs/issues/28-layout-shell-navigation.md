# Title

Layout shell, navigation, title screen, settings

# Summary

Build the player's structural UI per DESIGN §9.1: header with progress + settings menu, the five-section left navigation with visibility/badge logic, the title screen with content-warning gate, storage banners, and the responsive/a11y baseline every view issue inherits.

# Context

This is the frame all modality/mechanic views (29–39) plug into. It owns the shared a11y contract: focus management on view switch, the `aria-live` notification region (consumed by issue 37), landmark structure, and the tokens-only styling rule.

# Scope

- `packages/player/src/views/Shell.tsx` and `packages/player/src/components/{Header,Nav,TitleScreen,SettingsMenu,AboutDialog,Banner,Dialog,LiveRegion}.tsx` + CSS Modules; component tests alongside each file.

# Detailed Requirements

1. Structure: `<header>` (scenario title, chapter progress `2/4` from `progressSummary`, settings button) · `<nav aria-label>` (sections **Story / Evidence / Data / People / Questions** — chrome keys) · `<main>` rendering `activeView` · `LiveRegion`: visually-hidden `aria-live="polite"` element with exported context API `useLiveAnnouncer(): (message: string) => void` (consumed by issue 37).
2. Nav model — exact mapping table (single source for weaker agents):

| Section | Rows (source selector) | Click → `setActiveView` |
|---|---|---|
| Story | all chapters by `order`; unlocked normal, locked as lock-icon rows (count visible, **no title reveal**) | `{kind:"chapter", id}` (locked rows too — reader shows the locked placeholder) |
| Evidence | `visibleDocuments()` grouped by category, then `visibleImages()` | `{kind:"document", id}` / `{kind:"image", id}` |
| Data | one fixed "SQL console" row + `visibleTables()` | `{kind:"sql"}` / `{kind:"table", id}` |
| People | `visibleCharacters()` | `{kind:"character", id}` |
| Questions | one fixed row (+ "Accusation" row once `finalAccusation.chapter` unlocked; "Ending" row once solved) | `{kind:"questions"}` / `{kind:"accusation"}` / `{kind:"ending"}` |

   "New" dots: documents/images when id ∉ `readEvidence`; chapters/tables/characters when id ∉ session `seen` set (issue 25's `markSeen`, fired on `setActiveView` for that item). Hidden content is entirely absent; empty sections (no datapack, no characters) hide.
3. View switching: clicking a nav item sets `activeView` and `markSeen(kind, id)` where applicable; focus moves to the new view's `<h2>` (tabIndex −1 pattern); nav marks the active item with `aria-current="page"`.
4. Title screen (first load or after reset): title, description, estimated minutes, authors; when `contentWarnings` non-empty, warnings shown with a confirm button (`title.continue`) gating entry (§9.1); "continue playing" vs "start" label depends on existing progress.
5. Settings menu: language switcher (issue 27's component), progress reset entry (opens typed-confirmation dialog — learner must type the scenario title or `RESET`; wired to issue 26's `resetAndReload`), about entry.
6. About dialog: scenario title/version/authors, datastory-kit player version, MIT notice (§9.8).
7. Banners consume issue 26's exact `LoadResult` contract: `{kind:"unavailable"}` → persistent storage-unavailable banner (`progress.storageUnavailable`); `{kind:"versionMismatch", stored, current, salvage}` → banner offering reset (`clearProgress` + reload) or continue-at-own-risk (applies `salvage()` into the store). Both dismissible for the session, re-shown next load while the condition holds.
8. Responsive (§9.1): ≥1280 fixed side nav; 768–1279 collapsible (hamburger with `aria-expanded`); <768 single column best-effort (no layout crash; not a release gate).
9. A11y baseline: all interactive elements keyboard reachable in DOM order; focus ring from tokens; dialogs are focus-trapped with `Esc` close (small internal `Dialog` component — no dependency); color contrast AA against tokens (documented token pairs).
10. All strings via `useT()`; keys added to both catalogs (issue 27 rule).

# Acceptance Criteria

- [ ] Fixture walkthrough (component test): locked chapter shows as locked row without title; hidden table absent; after simulated unlock, item appears with badge.
- [ ] Content-warning fixture gates entry until confirmed; no-warnings fixture goes straight to title actions.
- [ ] View switch moves focus to the view heading (test via testing-library focus assertion).
- [ ] Reset dialog requires the exact typed phrase; wrong input keeps the button disabled.
- [ ] Both banners render under their simulated conditions and dismiss for the session.
- [ ] axe-core smoke on shell + title screen: no serious/critical violations.
- [ ] Bilingual: full shell renders in en and ja with no missing-key warnings (test mode).

# Validation

Component tests per above + axe run; manual keyboard-only walkthrough recorded in the PR checklist.

# Dependencies

- 24, 25, 26, 27 (26 is hard: banners and reset consume its `LoadResult`/`clearProgress` contract — dependency table updated accordingly)

# Non-goals

- The content of the five views (29–35), question UI (36), toasts (37 — the live region only is provided here), ending (39).

# Design References

- DESIGN.md §9.1 (shell), §9.8 (settings/notifications/reset), §7.4 (banners), §12 (mixed-lang rule)
