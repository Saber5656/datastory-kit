# ADR-006: Bilingual (ja/en) player chrome; one locale per scenario package

- Status: Accepted
- Date: 2026-07-10
- Deciders: Product owner (bilingual v1 UI + bilingual samples, 2026-07-10 requirements interview); packaging strategy by design agent

## Context

The user chose Japanese-and-English support in v1: the player UI must work in both languages and the sample scenario must ship in both. Scenario *content* (story prose, evidence, dialogue) is long-form authored text; the player *chrome* (buttons, labels, errors) is a small fixed string set. These have different translation economics.

## Decision

1. **Player chrome**: all user-visible strings live in locale catalogs `packages/player/src/i18n/ja.json` and `en.json` (flat dot-notation keys, ICU-style `{placeholder}` interpolation only — no plural rules needed in v1). A language switcher in the player settings toggles chrome locale at runtime; the default chrome locale follows the scenario's `locale`.
2. **Scenario content**: each scenario package declares exactly **one** `locale` (BCP 47: `ja` or `en` supported in v1) in `scenario.yaml`. Content is not inline-multilingual.
3. **Bilingual material** = two sibling scenario packages (e.g. `examples/school-library-case-ja`, `examples/school-library-case-en`). They may share image assets by duplication; the compiler treats them as independent scenarios with distinct ids (`…-ja`, `…-en`).
4. **Hard rule**: no hardcoded user-visible literals in player components; ESLint guard + review checklist enforce catalog usage.
5. Developer-facing documentation (README, docs/) is English; a Japanese authoring-guide translation is a v2 item.

## Consequences

Positive:

- Schema stays simple: content fields are plain strings, not `{ja:…, en:…}` maps — materially easier for weaker implementation agents and for authors.
- Chrome translation is a bounded, testable surface (catalog key parity check in CI).
- Adding a third language later = one catalog file + zero schema change.

Negative:

- Cross-locale content duplication is manual (a translation-sync tool is a v2 candidate).
- A learner cannot switch content language mid-play; they switch scenario. Documented in the deployment guide.

## Alternatives considered

- **Inline multilingual content fields**: doubles authoring effort per field, complicates validation ("which locales are complete?"), bloats bundles with unused text.
- **Chrome in Japanese only (i18n-ready structure)**: was the design agent's initial recommendation; the user explicitly upgraded to bilingual v1 — recorded as a scope decision.
- **Runtime translation service**: violates ADR-001 (no network) and privacy posture.
