# Title

Authoring guide

# Summary

Write `docs/guide/authoring.md`: the end-to-end creator manual — package layout, every file format with annotated examples, the unlock/question/hint design patterns, pedagogical guidelines, the generated diagnostics reference, and answer-writing rules for Japanese and English.

# Context

Authors are teachers and content creators, not necessarily engineers (DESIGN §2.1). The format is fully specified in DESIGN §5 for implementers; this guide re-teaches it for *creators* — task-oriented, example-first. It embeds the generated diagnostics table (issue 18) so error messages are searchable in one place.

# Scope

- `docs/guide/authoring.md` (English; ja translation is v2 per ADR-006) + example snippets; embed/regenerate hook for `docs/reference/diagnostics.md`.

# Detailed Requirements

1. Structure: Quick start (init → edit → validate → build → preview, mirroring §2.2) · Package tour (annotated §5.1 tree) · Manifest how-to · Writing chapters & gates (incl. the §7.3 invariants in author language: "exactly one start", "never gate a chapter on its own question", "never gate on the final accusation") · Documents & images (alt-text guidance, SVG/inline-image restrictions and why) · **Markdown safety rules** (§13.2 verbatim in author language: no raw HTML except `<ruby>/<rt>/<rp>`; no inline images/`<style>`/`<script>`/event handlers; `allowExternalLinks` behavior in both modes with the DS5101/DS5102 warnings; allowed protocols; what DS5001/DS5103 mean) · Data tables (CSV rules, types, filter/sort flags with UI screenshots, answer-leak warning from §6.3) · Characters & interviews (requires-chains, informational-only rule) · Questions, answers & hints · Publishing pointer to the deployment guide.
2. Answer-writing rules section (the §8/U3 knowledge, creator-voiced): list all reasonable variants (spacing, kana readings, name order for en); what normalization already forgives (width, case, katakana/hiragana, edge punctuation) with examples; the decoy `spoilerGuard` pattern; the "casual spoiler protection" honesty note (ADR-007).
3. Pedagogical guidelines (from §16.3, generalized): no modality lock-out rule with a self-check method; hint laddering (nudge → method → near-answer) with examples from the sample; reading-level and furigana guidance (`<ruby>` syntax snippet); estimated-minutes honesty.
4. Diagnostics reference: embed `docs/reference/diagnostics.md` by link + inline exactly these 10 codes with worked fixes: DS1001, DS2001, DS3001, DS3002, DS3009, DS4002, DS4003, DS5101, DS6001, DS7002.
5. Every YAML example must be an *excerpt of the actual sample scenario* (issue 40; issue 20 templates may be referenced when convenient but are not required) — kept honest by a docs test with exact fence syntax: fenced blocks opened as ` ```yaml verify=<scenario|images|datapack|character|questions> ` are extracted by `docs/tests/verify-examples.test.ts` (workspace test) and validated against the corresponding schema from `@datastory/schema`; each block must be a complete valid document for its schema (no partial excerpts in `verify` blocks — partial illustrations use plain ```yaml fences). Markdown/frontmatter examples are illustrative only (plain fences), stated in the guide.
6. Screenshots: 4–6 player screenshots (table browser flags in action, hint panel, ruby rendering) captured from the built ja sample once the player is complete (dependency on 42 guarantees this); stored under `docs/guide/img/` (≤200 KB each), with alt text.
7. Tone: plain English, sentence-case headings, no unexplained jargon; each section ends with a "checklist" box.

# Acceptance Criteria

- [ ] A first-time author following only the guide can go init→preview without touching DESIGN.md (validated by a fresh-eyes walkthrough recorded in the PR).
- [ ] All `verify`-tagged examples schema-validate in CI (test exists and a scratch-broken example fails it).
- [ ] Answer-rules section covers every normalization step of §8.1 with a creator-facing example.
- [ ] Diagnostics reference current (drift test from issue 18 already guards the table; guide links resolve).
- [ ] Screenshots present with alt text; guide passes markdownlint (add config if absent).

# Validation

Fresh-eyes walkthrough + CI example validation; content review against DESIGN §5/§8 for accuracy.

# Dependencies

- 18, 40, 42 (screenshots need the playable built sample; dependency table updated)

# Non-goals

- Japanese translation (v2, ADR-006); video tutorials; deployment specifics (issue 45).

# Design References

- DESIGN.md §2.2 (journey), §5 (format), §8 (answers), §16.3 (pedagogy), §11 (diagnostics)
