# Title

Progress persistence in localStorage

# Summary

Persist the engine's `Facts` to `localStorage` per DESIGN §7.4: debounced write-through, versioned schema, scenario-version compatibility handling, storage-unavailable fallback, and the reset flow's storage side.

# Context

Learners play across multiple sessions on shared school devices; progress loss is the product's worst failure short of unsolvable puzzles. §13.6 privacy rules apply: only content ids/counters/locale — never free-text input.

# Scope

- `packages/player/src/persist/progressStore.ts` + store wiring; unit tests with an injected storage stub. Exported API (exact):
  - `loadProgress(opts: {storage: Pick<Storage,"getItem"|"setItem"|"removeItem">, bundle: Bundle}): LoadResult`
  - `wireProgressPersistence(store: AppStore, bundle: Bundle, storage?: Storage): () => void` (returns unsubscribe; applies restored facts into the store on wire-up)
  - `clearProgress(storage: Storage, scenarioId: string): void` and `resetAndReload(storage: Storage, scenarioId: string): void`
  - Tests inject the stub via the `storage` parameter — no global monkey-patching.

# Detailed Requirements

1. Key: `datastory:v1:<scenarioId>`. Serialized shape per §7.4: `{schemaVersion: 1, scenarioVersion, correct: string[], solved, attempts, openedHints, askedTopics: string[], readEvidence: string[], chromeLocale, updatedAt}` (Sets ⇄ sorted arrays for stable serialization).
2. Zod schema for the stored value; `LoadResult` is a discriminated union consumed by issue 28's shell: `{kind:"fresh"}` · `{kind:"ok", facts, chromeLocale}` · `{kind:"schemaMismatch"}` · `{kind:"versionMismatch", stored: string, current: string, salvage(): Facts}` · `{kind:"unavailable"}` · `{kind:"corrupt"}` (parse/validation failure — treated like schemaMismatch but logged to console). Contract with the shell (§7.4 banner): `stored` = the persisted `scenarioVersion`, `current` = `bundle.scenario.version`; `salvage()` is invoked only when the learner chooses continue-at-own-risk; the reset choice calls `clearProgress`.
3. Version rule (§7.4): compare semver MAJOR of stored `scenarioVersion` vs bundle. Equal → ok. Different → `versionMismatch`; `salvage()` keeps only ids that resolve in the current bundle — resolution sources: question ids from `bundle.questions[].id` ∪ `bundle.finalAccusation.id`; evidence ids from `bundle.evidence.docs[].id` ∪ `bundle.evidence.images[].id`; topic keys as `${characters[].id}/${topics[].id}`; `attempts`/`openedHints` retained only for surviving question ids.
4. Save: subscribe to facts changes; debounce 250 ms; also flush on `visibilitychange→hidden` and `pagehide`. Serialize deterministically; guard payload < 64 KB (log + skip write if exceeded — should be impossible under content caps).
5. Unavailability: feature-detect with a write/remove probe at startup; on failure (Safari private mode, quota, disabled) → in-memory mode + expose `storageAvailable: false` for the banner (UI in issue 28, chrome key `progress.storageUnavailable`).
6. Quota errors during later writes downgrade to in-memory mode + banner (never crash).
7. `clear(scenarioId)` for the reset flow (UI/confirmation in issue 28/37 — this issue provides the primitive + a `resetAndReload()` helper).
8. `chromeLocale` persists here too (consumed by issue 27).
9. Never persist: learner free-text answers, timestamps beyond `updatedAt`, device info (§13.6) — asserted by a serializer test on key allowlist.

# Acceptance Criteria

- [ ] Round-trip: mutate facts → debounce flush → reload from stub → identical facts (Sets restored).
- [ ] Each `load()` result kind covered by a test, incl. `salvage()` dropping unknown ids and keeping valid ones.
- [ ] Debounce collapses rapid mutations to one write (spy count); `pagehide` forces flush.
- [ ] Unavailable and mid-session-quota paths degrade to memory mode without exceptions.
- [ ] Serializer key-allowlist test passes (no extra keys ever written).
- [ ] Payload for a maxed-out synthetic scenario (40 questions × attempts, 40 evidence, 240 topics) stays < 64 KB.

# Validation

Unit tests with stubbed storage; issue 42's E2E covers real-browser reload persistence and the reset flow.

# Dependencies

- 25

# Non-goals

- Reset confirmation UI and version-mismatch banner UI (28), export/import (v2), cross-device sync (v2).

# Design References

- DESIGN.md §7.4 (persistence), §13.6 (privacy), §9.8 (reset UX)
