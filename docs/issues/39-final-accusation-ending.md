# Title

Final accusation flow and ending screens

# Summary

Implement the climax per DESIGN §9.7: the visually distinct accusation form, the solved transition, epilogue unlock, and the ending screen with a non-judgmental play summary and replay pointer.

# Context

The accusation is the emotional payoff the whole material builds toward. It reuses the judging machinery (issue 36) but gets bespoke staging: a deliberate, weighty submission moment; a celebratory-but-calm success; and a retry path that never humiliates (school audience, unlimited attempts).

# Scope

- `packages/player/src/views/AccusationView.tsx`, `packages/player/src/views/EndingScreen.tsx` + CSS Modules; component tests alongside (`AccusationView.test.tsx`, `EndingScreen.test.tsx`).

# Detailed Requirements

1. Entry: a distinct nav item under Questions (`accuse.navLabel`) visible once `finalAccusation.chapter` is unlocked; question cards' pointer (issue 36) routes here.
2. Accusation form: case-file styling (tokens; bordered document look), `promptHtml` via the shared `SafeHtml`, input per type (§5.8: text in the sample; choice supported), explicit two-step submit — button `accuse.submit` opens an inline confirm (`accuse.confirm {answer}`) then `accuse.confirmYes` judges via `useJudge`. Confirm-echo semantics: for `text`, `{answer}` is the raw entered string rendered **only as an escaped React text node** (never `SafeHtml`), and judging receives that raw string (normalization happens inside `useJudge`); for `choice`, `{answer}` displays the selected option's `label` while judging receives the option `id`.
3. Incorrect: `accuse.retry` copy (distinct from regular questions, references reviewing evidence + hints); attempts counted; hints available via issue 38's panel (dependency); no cooldown.
4. Correct: `solved` flips (engine) → full-screen solved transition (calm celebratory panel, `prefers-reduced-motion` honored) → `explanationHtml` (the case resolution) → continue button to the ending screen. Epilogue chapters are now unlocked; toast suppression per issue 37.
5. Ending screen: scenario title + `ending.solvedHeading`; neutral summary table with exact formulas from engine facts — chapters completed = count of chapters where `isChapterComplete` (§7.1) is true; questions answered = `facts.correct.size + 1` (the final); total attempts = `sum(Object.values(facts.attempts))`; hints opened = `sum(Object.values(facts.openedHints))` — phrasing `ending.summaryNote` explicitly non-evaluative ("how you investigated, not a score"); links: read epilogue chapter(s) (primary), revisit evidence, `ending.replayNote` pointing at settings→reset for replay.
6. Post-solve app state: everything stays explorable (evidence, transcripts, questions show explanations); the ending screen re-reachable from nav (`ending.navLabel` appears once solved).
7. Learner's accusation text is never persisted (§13.6) — echo lives only in the transient confirm.
8. Strings in both catalogs; the solved transition is announced assertively (`role="alert"` once).

# Acceptance Criteria

- [ ] Full flow on fixture: wrong accusation → retry copy + attempt count; correct → solved panel → explanation → ending screen whose four summary numbers equal the formulas above computed from the test's known facts.
- [ ] XSS-shaped text input (`<img src=x onerror=…>`) renders escaped in the confirm echo, executes nothing, and appears nowhere in localStorage after the flow (negative asserts).
- [ ] Epilogue chapter becomes readable (issue 29 integration) and ending is re-reachable from nav.
- [ ] Two-step confirm required; `Esc`/cancel returns to editable form; echoed text absent from DOM after flow (negative assertion).
- [ ] Reload after solve restores solved state directly (no re-transition animation on load).
- [ ] Reduced-motion path verified; axe smoke + bilingual clean.

# Validation

Component tests per above (`pnpm --filter @datastory/player test -- AccusationView EndingScreen`) + axe/strict-i18n checks. (Narrative pacing review is product-level validation owned by issues 40/42, not a gate for this issue.)

# Dependencies

- 36, 37, 38 (dependency table updated — the retry path references the hint panel)

# Non-goals

- Score/grade output, shareable results cards (v2 if ever); multiple-ending branching (v1 format has exactly one accusation, §5.8).

# Design References

- DESIGN.md §9.7 (accusation/ending), §7.2 (solved transition), §5.8 (finalAccusation shape), §13.6 (no input persistence)
