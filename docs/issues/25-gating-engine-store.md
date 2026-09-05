# Title

Gating engine and app state store

# Summary

Implement the pure-TypeScript gating engine (DESIGN §7.1–§7.2) — persisted facts in, derived unlock state out — and wrap it in the Zustand app store that all views consume.

# Context

The engine encodes the game's core rules. It is deliberately framework-free (testable as a table of transitions) and deliberately trusting: bundles are always compiler-emitted, and DESIGN §7.3's compile-time invariants guarantee the graph is sound, so the engine implements no cycle/reachability analysis (semantic dependency on issue 15's guarantees, not a build dependency). Defensive rule for robustness: an `afterQuestions` entry or `unlockedBy` value that resolves to no known id is treated as never-satisfied/never-visible — no crash, no unlock. The store is a thin binding; UI issues (28–39) subscribe to selectors, never recompute rules.

# Scope

- `src/engine/gating.ts`, `src/engine/types.ts`, `src/store/appStore.ts` in `packages/player`; exhaustive unit tests.

# Detailed Requirements

1. Persisted-facts type `Facts` per §7.1: `correct: Set<string>`, `solved: boolean`, `attempts: Record<string, number>`, `openedHints: Record<string, number>`, `askedTopics: Set<string>` (key format `charId/topicId`), `readEvidence: Set<string>`.
2. Pure derivations (all `(bundle, facts) => …`, memo-friendly):
   - `isChapterUnlocked(ch)` per §7.1 (start / afterQuestions ⊆ correct / solved).
   - `visibleDocuments/Images/Tables/Characters()` filtered by their `unlockedBy` chapter's unlock state.
   - `isTopicAskable(char, topic)` (character visible ∧ requires ⊆ asked-of-that-character).
   - `isChapterComplete(ch)`; `progressSummary()` → `{unlockedChapters, totalChapters, answeredQuestions, totalQuestions}`.
3. Transition functions per §7.2 (pure: `(facts, event) => {facts, newlyUnlocked}`):
   - `answerCorrect(questionId)` — adds to `correct` (or sets `solved` for the final id), returns the exact DESIGN §7.2 batch type `NewlyUnlocked = {reason: "question"|"finalAccusation", chapters: string[], documents: string[], images: string[], tables: string[], characters: string[]}` computed as a before/after visibility diff.
   - `recordAttempt(questionId)`; `openHint(questionId, tier)` — if `tier > current + 1` it is a no-op returning the same facts reference, otherwise `openedHints[q] = max(current, tier)` (resolves the §7.2 max-vs-order wording: strict order enforced by the guard, max applied within it); `askTopic(charId, topicId)` (no-op unless askable), `markEvidenceRead(id)`, `reset()`.
4. Zustand store (`packages/player/src/store/appStore.ts`) holding `bundle`, `facts`, `activeView` (`{kind: "chapter"|"document"|"image"|"sql"|"table"|"character"|"questions"|"accusation"|"ending", id?: string}`), `pendingNotifications: NewlyUnlocked[]`, and session-only `seen: Record<"chapter"|"table"|"character", Set<string>>` (never persisted — §7.4 stores evidence read-state only). Exported actions (exact signatures): `answerCorrect(questionId: string)`, `recordAttempt(questionId: string)`, `openHint(questionId: string, tier: number)`, `askTopic(charId: string, topicId: string)`, `markEvidenceRead(id: string)`, `markSeen(kind, id)`, `setActiveView(v)`, `consumeNotifications(): NewlyUnlocked[]`, `reset()`, plus `subscribeFacts(listener: (facts: Facts) => void): () => void` for persistence (issue 26).
5. Privacy guard (DESIGN §13.6, non-skippable): `Facts` and everything reachable from `subscribeFacts` may contain only content ids, counters, booleans, and sets of ids — never learner-typed answer text or option labels; a serializer key/type-allowlist test enforces this.
5. Engine must be UI-import-free (`eslint no-restricted-imports`: no `react`, no `*.tsx` from `src/engine/**`).
6. Determinism: given same bundle+facts, all derivations referentially stable enough for React (memoize per facts-revision counter).

# Acceptance Criteria

- [ ] Transition-table test covers every row of DESIGN §7.2 on the fixture bundle (answer→unlock cascade, hint tier ordering incl. skip attempt, topic requires chain, evidence read, reset).
- [ ] `newlyUnlocked` reports exactly the delta with `reason: "question"` (fixture: one correct answer unlocks 1 chapter + 2 documents + 1 table → asserted lists).
- [ ] Final-question correctness sets `solved`, unlocks `solved` chapters, and the batch carries `reason: "finalAccusation"`.
- [ ] `openHint` with tier 2 before tier 1 is a no-op (facts unchanged, same reference).
- [ ] Privacy allowlist test passes; a scratch field carrying a string set fails it.
- [ ] Engine files import no React (lint + grep test).
- [ ] 100% branch coverage on `gating.ts` (it is small and load-bearing).

# Validation

Unit tests per above; the same fixture playthrough script is reused by issue 42's E2E to keep engine and UI honest against each other.

# Dependencies

- 05, 09 (types), 24 (package + bundle types)

# Non-goals

- Persistence I/O (26); any rendering (28+); judging/hash comparison (36 — the engine receives *already-judged* correctness).

# Design References

- DESIGN.md §7.1–§7.2 (model + transitions), §7.3 (invariants it may assume), §9.8 (notification consumer)
