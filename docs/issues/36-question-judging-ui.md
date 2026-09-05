# Title

Question forms and client-side judging

# Summary

Implement the Questions view per DESIGN §9.7 and the judging flow of §8.3: choice/text forms, hash-based verification via the shared normalizer, correct/incorrect feedback with explanation reveal, and engine integration for attempts and unlocks.

# Context

This is where the DESIGN §8 contract becomes user-facing: the same `normalizeAnswer`+`hashAnswer` code the compiler used (issue 10) judges the learner's input, so tolerant matching works identically. Feedback tone matters — school audience, unlimited retries, no shaming (§5.8, §9.7).

# Scope

- `packages/player/src/views/QuestionsView.tsx`, `packages/player/src/views/QuestionCard.tsx`, `packages/player/src/views/useJudge.ts` + CSS Modules; component tests alongside.

# Detailed Requirements

1. Questions view: gate questions of unlocked chapters grouped by chapter (chapter order); the final accusation is **excluded** here (issue 39 owns it, with a pointer card once its chapter is unlocked). Supports `activeView.id` anchor from issue 29 (scroll + focus that card).
2. `QuestionCard` states: unanswered (form) · answered-correct (✓, learner's matched answer *not* echoed for text — show `question.solved` chrome + `explanationHtml` rendered only through the shared `SafeHtml`, as are `promptHtml` and hint HTML: no new `dangerouslySetInnerHTML` call sites, DESIGN §13.2 boundary 3) · judging (async hash in flight, ≤ a frame or two — still guard double-submit).
3. Forms: `choice` → radiogroup (labels via option `label`), submit disabled until selection; `text` → single-line input (`autocomplete=off`, `spellcheck=false`), submit disabled when empty/whitespace (empty never counts as an attempt, §8.3).
4. `useJudge(question)`: `submit(value)` → `recordAttempt` (engine) → `hashAnswer(bundle.salt, value or optionId)` → compare to `answerHashes` → on match call the store's `answerCorrect(questionId)`, which queues the `NewlyUnlocked` batch into `pendingNotifications` (issue 25). This issue does **not** render notifications — issue 37 consumes the queue; until 37 lands the queue simply accumulates (invisible, harmless).
5. Incorrect feedback: `question.retry` message (varies by attempt count: gentle escalation at 3+ attempts nudging hints — `question.retryHint`), input preserved, `role="status"` announcement. Never reveal accepted answers or "how close" the input was.
6. Attempt counter display: quiet (`question.attempts {n}` small text); explanations render only after correct (§5.8).
7. Free-text input is never persisted (§13.6) — only attempt counts (engine facts).
8. Hints entry point per card: this issue renders only a stub button with stable test id `hint-entry-{questionId}` and an aria-label from the catalog, marked disabled — issue 38 (which depends on this issue) replaces the stub with the real panel wiring.
9. Strings in both catalogs; forms fully labeled; error/status regions announced.

# Acceptance Criteria

- [ ] Text question fixture (the load-bearing golden-vector parity assertion): the fixture authors `accepted: ["さとうりく"]`; learner input `"ｻﾄｳﾘｸ"` (half-width katakana, no space) is judged **correct** (NFKC → katakana → hiragana fold); learner input `"  さとう　りく  "` with inner ideographic space is judged **incorrect** (whitespace collapses to one space, which does not match the spaceless accepted form) — both asserted, proving the §8.1 pipeline end-to-end without implying kana→kanji conversion.
- [ ] Choice flow: select → submit → correct/incorrect paths; double-submit guarded.
- [ ] Empty input: submit disabled; attempt count unchanged.
- [ ] Incorrect at 1st vs 3rd attempt shows escalating copy; correct reveals explanation and queues the batch (store assertion on `pendingNotifications`).
- [ ] Answered-correct state survives reload (facts round-trip via issue 26's storage stub — 26 is a dependency).
- [ ] No accepted-answer plaintext anywhere in DOM at any state (negative assertion).
- [ ] axe smoke + bilingual clean.

# Validation

Component tests per above; cross-browser hash behavior re-verified in issue 42 (WebKit).

# Dependencies

- 10, 25, 26, 28 (dependency table updated; 37/38 build on this issue, not vice versa)

# Non-goals

- Final accusation UX (39); hint content UI (38); per-question time tracking (never — privacy posture).

# Design References

- DESIGN.md §9.7 (question UX), §8.1–§8.3 (judging), §5.8 (retry/explanation rules), §13.6 (no input persistence)
