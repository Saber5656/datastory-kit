# Title

Tiered hint system

# Summary

Implement the hint UI per DESIGN §9.7: per-question tiered reveal (1 → n, strictly in order), first-open confirmation, persistent opened state, and the hint entry points wired from question cards and empty-result nudges.

# Context

Hints are the teacher-load reducer the user explicitly pulled into v1: authors pre-stage graduated nudges (the nudge → method → near-answer ladder is content guidance from DESIGN §16.2; §5.8 defines the shape/caps) so stuck learners self-serve. The design tension is availability without temptation — hence the confirmation step and quiet styling.

# Scope

- `packages/player/src/views/HintPanel.tsx` + replacing issue 36's stub hint button (`hint-entry-{questionId}`) with the real wiring; component tests alongside. (The table browser's `table.hintNudge` is plain navigation owned by issue 34 — nothing to wire here.)

# Detailed Requirements

1. Per question with `hints.length > 0`, per DESIGN §9.7: **per-tier buttons** `ヒント1 … n` (`hints.tierLabel {k}`); tier k's button is enabled only after tier k−1 is opened; future tiers beyond k+1 render disabled (visible count is authored and public — only content is gated). Questions without hints show nothing.
2. Panel (inline expansion under the card, not a modal): opened tiers listed with their compiled sanitized `hints[k].html` rendered only through the shared `SafeHtml` (the player receives HTML, not Markdown); tiers strictly sequential (engine `openHint` enforces; UI never enables tier k+2).
3. First-open confirmation per question (not per tier): inline confirm (`hints.confirmOpen` text + confirm/cancel buttons) shown before the first tier opens; afterwards tiers open directly. Confirmation applies when `(openedHints[q] ?? 0) === 0` (missing key = zero; no extra persistence).
4. Opened hints persist (facts, §7.1) and stay expanded-available after correct answers (post-solve review value) — collapsed by default once answered.
5. Solved question: hints section renders collapsed with `hints.viewAfterSolve` toggle.
6. No penalties, scores, or judgmental copy anywhere; the ending summary (issue 39) reports hint usage neutrally.
7. Strings in both catalogs; the reveal is announced (`role="status"`: `hints.revealed {k}`).

# Acceptance Criteria

- [ ] Tier flow: confirm → tier 1 opens → tier 2 button enables → tier 2; tier 3 stays disabled until 2 opens (per-tier buttons per §9.7; engine no-op path also UI-guarded).
- [ ] Opened tiers survive reload (issue 26 stub round-trip) and render expanded pre-solve, collapsed post-solve with working toggle.
- [ ] Question without hints renders no hint UI.
- [ ] Confirmation appears exactly once per question.
- [ ] Compiled sanitized hint `html` fixture (containing `<strong>`) renders via `SafeHtml` only; no plaintext answer leakage assertions reuse issue 36's negative check.
- [ ] axe smoke + bilingual clean.

# Validation

Component tests per above; pedagogy tone review alongside the sample scenario (issue 40).

# Dependencies

- 36

# Non-goals

- Adaptive/auto-suggested hints (v2, would need heuristics); global hint budget mechanics; teacher-side hint analytics (no telemetry — §13.6).

# Design References

- DESIGN.md §9.7 (hint UX), §5.8 (authoring shape), §7.1–§7.2 (openedHints)
