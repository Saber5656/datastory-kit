# Title

Player scaffold and bundle loader

# Summary

Create `packages/player` (React 18 + Vite + TypeScript per ADR-004): app entry, the `story.json` bundle loader with schema validation, the shared `SafeHtml` component, error boundary, and the CI bundle-size budget.

# Context

The player consumes the compiler↔player contract (DESIGN §6.2); validating the bundle at load time turns emitter drift into an immediate, visible error instead of undefined runtime behavior. ADR-004 constraints apply: no router, CSS Modules, relative asset paths so bundles work at any base path.

# Scope

- Package scaffold: Vite config, `index.html` shell, `src/main.tsx`, `src/App.tsx` placeholder.
- `src/bundle/loadBundle.ts`, `src/components/SafeHtml.tsx`, `src/components/ErrorScreen.tsx`, `src/styles/tokens.css`.
- Size-budget wiring in CI.

# Detailed Requirements

1. Vite config: `base: "./"` (relative assets, §6.1); `build.target: "es2022"`; output `dist/`. Required outcome (CSP §13.3): the emitted `index.html` contains only `src`-referenced external scripts and no inline `<script>` bodies or inline event handlers — configure whatever Vite options are needed to achieve it and enforce it with the build-grep test below.
2. `index.html`: minimal — `<div id="root">`, module script, **no** CSP meta (the CLI injects it at build assembly, issue 22; a comment marks the injection point), `<noscript>` message.
3. `loadBundle(url = "story.json"): Promise<Bundle>`: fetch (relative URL), parse, validate with the compiled Zod schemas from `@datastory/schema` (issues 05–09) plus top-level shape (§6.2); on failure render `ErrorScreen` with a non-technical message (chrome string) + a details disclosure containing the technical error. Check `formatVersion === 1` and `compiler.name` presence; version mismatch gets its own message.
4. `SafeHtml`: the **only** `dangerouslySetInnerHTML` call site (DESIGN §13.2); props `{html: string, as?: keyof JSX.IntrinsicElements}`; ESLint rule (`react/no-danger` allowed only in this file via override) enforced from this issue on.
5. `tokens.css`: CSS custom properties — color palette (paper-tone background, ink text, accent, success/error), spacing scale, type scale (min font 14px), focus-ring style; light theme only in v1; all later components must use tokens (reviewed per issue).
6. Error boundary at app root → `ErrorScreen` (reload button; no telemetry — §13.6).
7. Dev affordance: `pnpm --filter @datastory/player dev` serves the app with a fixture bundle from `public/` (hand-written `story.json` fixture checked in until issue 40).
8. Size budget (DESIGN §15): CI step (extend issue 02 workflow) asserting gzipped `dist/assets/*.js + *.css` total ≤ 300 KB (script: `scripts/check-size.mjs`; sql.js wasm excluded).
9. Publish config: `"files": ["dist"]`; package exports nothing importable except metadata (the dist is consumed by the CLI as static files).

# Acceptance Criteria

- [ ] `pnpm --filter @datastory/player build` emits `dist/` with only hashed asset filenames + `index.html`; no inline `<script>` bodies (test greps built html).
- [ ] Loader accepts the `valid-full` compiled fixture story.json; rejects a mutated one (missing `chapters`) with `ErrorScreen` rendered (component test).
- [ ] `formatVersion: 2` fixture shows the version-mismatch message.
- [ ] `SafeHtml` renders pre-sanitized `*Html` fixture content **without modification** (no runtime sanitizer is added — DESIGN §13.2 assigns sanitization to compile time); ESLint fails when `dangerouslySetInnerHTML` is used elsewhere (verified by scratch test).
- [ ] Size check passes and fails correctly (scratch: import a large lib, watch it fail, revert).
- [ ] Relative-asset smoke: `pnpm --filter @datastory/player build` then serve `dist/` from a nested path with any static server — no absolute-path 404s (grep `dist/index.html` for leading-`/` URLs as the mechanical check; full deployed `--base-path` behavior is validated in issues 22/42).

# Validation

Vitest component tests + build greps; manual dev-server run with the fixture bundle showing the placeholder App reading scenario title.

# Dependencies

- 01, 04, 11 (issue 11 transitively requires 05–10, so all compiled schemas and types are available through the barrel)

# Non-goals

- Any real UI (28+); worker (32); i18n provider (27 — `ErrorScreen` may hardcode en/ja pair temporarily with a TODO to migrate).

# Design References

- DESIGN.md §4.3 (runtime architecture), §6.2 (bundle contract), §13.2/§13.3 (SafeHtml/CSP split), §15 (budget) · ADR-004
