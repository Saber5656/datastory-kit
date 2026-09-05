# Title

Player i18n runtime with ja/en catalogs

# Summary

Implement the chrome i18n layer per DESIGN §12 and ADR-006: flat-key JSON catalogs for Japanese and English, a `useT()` hook with `{placeholder}` interpolation, locale switching persisted per scenario, and the CI key-parity check.

# Context

The user explicitly chose bilingual v1 chrome. Scenario *content* stays single-locale per package (ADR-006); this layer covers only player-owned strings. Every subsequent UI issue depends on this hook — no hardcoded user-visible literals are allowed from here on.

# Scope

- `packages/player/src/i18n/ja.json`, `packages/player/src/i18n/en.json`, `packages/player/src/i18n/index.tsx` (provider + hooks), ESLint guard, key-parity test.

# Detailed Requirements

1. Catalogs: flat dot-notation keys grouped by area (`nav.*`, `title.*`, `chapter.*`, `evidence.*`, `sql.*`, `table.*`, `interview.*`, `question.*`, `hints.*`, `accuse.*`, `ending.*`, `settings.*`, `progress.*`, `notify.*`, `error.*`). This issue seeds the following required keys (en + ja values + params documented in the file): `nav.story`, `nav.evidence`, `nav.data`, `nav.people`, `nav.questions`, `title.start`, `title.continue`, `settings.language`, `settings.reset`, `settings.about`, `progress.storageUnavailable`, `progress.versionMismatch`, `error.bundleLoad`, `error.bundleVersion`, `error.reload`. Later UI issues extend both files **in the same PR** as their components (each issue states this; the parity test enforces it mechanically).
2. `I18nProvider` props `{scenarioLocale, initial?: Locale}`: resolution order = persisted `chromeLocale` (issue 26) → `scenarioLocale`. Exposes `locale`, `setLocale` (persists via issue 26), `t(key, params?)`, and `Intl`-backed formatting helpers `formatNumber(n)` / `formatDate(d)` bound to the chrome locale (DESIGN §12 — used for counts and elapsed-ms displays).
3. `t()` behavior: exact-key lookup in active catalog → fallback to `en` + one `console.warn` per missing key per session → last resort returns the key literally. Interpolation replaces `{name}` tokens from `params`; unreplaced tokens left visible (debuggable).
4. Dev/test strictness: in Vitest, a missing key **fails** (provider test mode throws) — components cannot ship untranslated.
5. `lang` propagation (§12 rule — chrome *elements* carry `lang`, never a subtree wrapper that would mislabel scenario content): export `chromeLangProps(): {lang?: string}` returning `{lang: chromeLocale}` only when chrome locale ≠ scenario locale, plus a `<ChromeText>` convenience component applying it to a `<span>`. Scenario content always remains under `<html lang={scenario.locale}>` untouched.
6. Key-parity test: `ja.json` and `en.json` key sets identical; no empty values; interpolation tokens per key identical across locales.
7. ESLint guard: `react/jsx-no-literals` scoped to `packages/player/src/{components,views}` **plus** a check on user-visible string props (`aria-label`, `title`, `placeholder`, `alt` passed as literals — via `jsx-no-literals`' `noAttributeStrings` or an equivalent no-restricted-syntax rule), with per-line opt-outs requiring a justification comment.
8. Language switcher *component* (used by issue 28's settings menu): accessible `<select>` labeled by `settings.language`, options "日本語"/"English" (each rendered in its own language + `lang` attr).

# Acceptance Criteria

- [ ] Switching locale re-renders visible chrome instantly (component test on a fixture view) and persists (stub storage assert).
- [ ] Default resolution follows persisted → scenario-locale order (tests for both paths).
- [ ] Missing-key behavior: warn+fallback in prod mode, throw in test mode (both asserted).
- [ ] Key-parity + token-parity test green; a deliberate mismatch fails (scratch-verified).
- [ ] Lint guard fails on both a scratch JSX text literal and a scratch literal `aria-label` (two scratch checks, reverted).
- [ ] Interpolation: `t("question.attempts", {n: 3})` renders per catalog for both locales; `formatNumber(1234)` differs appropriately between locales in a test.

# Validation

Unit/component tests per above; manual bilingual click-through once issue 28 lands.

# Dependencies

- 24, 26 (persisted `chromeLocale` is part of this issue's contract, so 26 is a hard dependency — dependency table updated accordingly)

# Non-goals

- Scenario-content translation (ADR-006: separate packages); plural rules/ICU MessageFormat (not needed v1); RTL (v2 with new locales).

# Design References

- DESIGN.md §12 (i18n), §9.8 (switcher placement) · ADR-006
