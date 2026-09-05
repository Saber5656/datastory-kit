# ADR-002: TypeScript everywhere, pnpm workspaces monorepo

- Status: Accepted
- Date: 2026-07-10
- Deciders: Product owner (TypeScript decision, 2026-07-10 requirements interview); package manager and layout chosen by design agent

## Context

The product has four tightly coupled deliverables: a scenario schema, a compiler, a browser player, and a CLI. They share type definitions (the scenario format and the compiled bundle format are contracts between compiler and player). The runtime is browser JavaScript, so a single-language stack avoids duplicated schema definitions. The user chose TypeScript for both CLI and runtime.

## Decision

1. **Language**: TypeScript (strict mode) for all packages. Node.js `>=20` for tooling.
2. **Repository shape**: a single monorepo managed with **pnpm workspaces** (pnpm pinned via the `packageManager` field / Corepack).
3. **Packages** (npm scope `@datastory/*`):

| Path | npm name | Purpose |
|---|---|---|
| `packages/schema` | `@datastory/schema` | Zod schemas, shared TS types, JSON Schema export. No I/O. |
| `packages/compiler` | `@datastory/compiler` | Scenario package → compiled bundle (Node-side). |
| `packages/player` | `@datastory/player` | React static player app; ships prebuilt `dist/` assets. |
| `packages/cli` | `@datastory/cli` (bin: `datastory`) | init / validate / build / preview. |
| `examples/*` | not published | Sample scenario packages (ja, en). |

4. **Task running**: plain `pnpm -r` scripts (`build`, `test`, `lint`, `typecheck`). No Turbo/Nx in v1.
5. **Testing**: Vitest for unit/component tests, Playwright for end-to-end.
6. **Lint/format**: ESLint (typescript-eslint) + Prettier, enforced in CI.

## Consequences

Positive:

- One schema definition (Zod) serves the compiler (validation), the player (types), and editors (exported JSON Schema).
- Atomic cross-package changes in one PR; no version skew between compiler and player.
- pnpm's strict `node_modules` prevents phantom dependencies — relevant to the supply-chain posture (DESIGN.md §13.7).

Negative:

- Contributors need Corepack/pnpm awareness (mitigated: one-line setup in README).
- Publishing multiple npm packages requires a coordinated release workflow (issue 46).

## Alternatives considered

- **npm workspaces**: workable, but weaker dependency isolation and slower installs; no strictness benefit.
- **Separate repositories per package**: rejected — the scenario format contract would drift between repos.
- **Python CLI + TS runtime**: rejected by the user in the requirements interview (dual-language schema duplication).
- **Turbo/Nx**: unnecessary at this package count; adds tooling surface.
