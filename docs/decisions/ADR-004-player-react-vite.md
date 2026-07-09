# ADR-004: Player built with React 18 + Vite + Zustand, styled with CSS Modules

- Status: Accepted
- Date: 2026-07-10
- Deciders: Design agent (conservative default; user delegated runtime detail decisions)

## Context

The player is a single-page static app with moderate interactivity (evidence browser, SQL console, dialogue UI, gated progression). It will be implemented largely by lower-capability coding agents executing granular issues, and later maintained by open-source contributors. Framework choice therefore optimizes for: (a) implementation-agent familiarity, (b) contributor pool size, (c) testing maturity, (d) acceptable bundle size — in that order.

## Decision

1. **UI framework**: React 18 with function components + hooks only.
2. **Build tool**: Vite (`@vitejs/plugin-react`), TypeScript strict.
3. **State**: Zustand for the single app store (gating/progress state), React context for i18n. No Redux.
4. **Styling**: CSS Modules with a small design-token stylesheet (`tokens.css`). No Tailwind, no CSS-in-JS runtime.
5. **Routing**: no router library; the player is a single view with client-side panel switching driven by store state (the build output must work from any base path, including `file://`-adjacent static servers, so URL routing is deliberately avoided in v1).
6. **Component testing**: Vitest + @testing-library/react; E2E: Playwright.

## Consequences

Positive:

- React is the most reliably known framework for both human contributors and implementation agents — fewer mis-implemented issues.
- No router and no CSS framework keeps the dependency tree (and supply-chain surface, DESIGN.md §13.7) small.
- Zustand keeps the gating engine testable as a plain TS module with the store as a thin wrapper.

Negative:

- Runtime payload ~45 KB gzipped (React+DOM) — larger than Svelte/Preact but dwarfed by sql.js WASM; within the performance budget (DESIGN.md §15).
- No URL deep-linking to a specific evidence item in v1 (acceptable; v2 candidate).

## Alternatives considered

- **Svelte**: smaller output, but smaller contributor/agent familiarity; riskier mechanical implementation.
- **Preact**: compat pitfalls with testing-library and ecosystem types outweigh the size win here.
- **Vanilla TS**: highest control, but the dialogue/gating UIs become hand-rolled state-DOM sync code — the classic source of subtle bugs for weaker implementation agents.
- **Tailwind CSS**: fast to build with, but adds a build-time dependency and utility-class churn in diffs; CSS Modules + tokens is sufficient at this UI scale.
