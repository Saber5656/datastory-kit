# Title

Character interviews UI

# Summary

Implement the People view per DESIGN §9.6: character grid, per-character interview screen with topic list gated by `requires`, and a persistent re-readable transcript of asked topics.

# Context

Interviews carry testimony and personality — the narrative warmth of the product. Mechanically they are simple (informational-only, §3.2): asking a topic reveals its authored reply and may make dependent topics askable. State lives in `askedTopics` (§7.1) so transcripts survive reloads.

# Scope

- `packages/player/src/views/CharacterGrid.tsx`, `packages/player/src/views/InterviewScreen.tsx` + CSS Modules; component tests alongside.

# Detailed Requirements

1. Grid: visible characters as cards — portrait (or initials avatar generated from `name` on the token palette), name, role; "new" badge when the character has ≥1 newly-askable topic not yet asked (derive: askable ∧ ¬asked).
2. Interview screen: header (portrait, name, role); transcript region renders asked topics in **authored topic order** (filtered to `askedTopics` — §7.1 stores a Set, so no chronology exists or is implied; chrome copy must not suggest time order): learner's question line (topic `label`, styled as the player speaking) then the reply (`topic.replyHtml` rendered only through the shared `SafeHtml`); then the topic chooser: currently askable & unasked topics as buttons.
3. Intro state: until the character has ≥ 1 asked topic, show `interview.intro` above the root-topic buttons; it disappears permanently for that character after the first ask.
4. Topics with unmet `requires` are **hidden** entirely (§9.6 — no teasers); when *asking a topic* satisfies another topic's `requires` (via `askedTopics` — replies themselves never unlock anything, §3.2/§5.7), the new topic button appears immediately (subtle `interview.newTopic` inline note).
5. Asking fires `askTopic` (engine; idempotent) — no other side effects exist by design (§3.2).
6. All-asked state: `interview.noMoreTopics` closing line ("今は他に聞けることはなさそうだ。" / en equivalent).
7. Back to grid affordance; focus management per §9.1 (focus to character name heading on open).
8. Strings in both catalogs; transcript readable by screen readers as a conversation (each turn labelled with speaker via visually-hidden text).

# Acceptance Criteria

- [ ] Fixture: hidden topic appears only after its `requires` chain is asked; transcript accumulates and re-renders identically after simulated reload (facts round-trip via issue 26 stub).
- [ ] Character with no asked topics shows intro state + all root topics; all-asked shows the closing line.
- [ ] "New" badge logic verified pre/post asking; badge clears appropriately.
- [ ] A fixture `replyHtml` containing compiled `<strong>` renders through `SafeHtml` (no new `dangerouslySetInnerHTML` call sites).
- [ ] Intro state appears pre-ask and is gone post-ask.
- [ ] axe smoke + ja/en strict-mode clean; keyboard-only interview possible.

# Validation

`pnpm --filter @datastory/player test -- CharacterGrid InterviewScreen` with a small interview fixture + axe/i18n checks — fully local to this issue (sample-scenario tone review belongs to issue 40).

# Dependencies

- 25, 26, 28

# Non-goals

- Free-text questioning, LLM characters (v2 explicitly out, §3.2/§3.3); dialogue-triggered unlocks (v2); typing animations.

# Design References

- DESIGN.md §9.6 (interviews), §5.7 (topic model), §7.1–§7.2 (askedTopics), §3.2 (informational-only)
