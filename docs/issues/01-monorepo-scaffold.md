# Title

Monorepo scaffold and tooling baseline

# Summary

Create the pnpm-workspaces TypeScript monorepo skeleton: root configs (TypeScript strict base, ESLint, Prettier, Vitest), workspace layout, MIT license, and repository hygiene files. No feature packages yet — this issue delivers the ground every other issue builds on.

# Context

DESIGN.md §4.1 fixes the repository layout and ADR-002 fixes the toolchain (TypeScript strict, pnpm workspaces, Vitest, ESLint+Prettier, Node ≥ 20, plain `pnpm -r` scripts — no Turbo/Nx). Feature packages (`schema`, `compiler`, `player`, `cli`) are scaffolded by their own issues (04, 12, 24, 19); this issue must leave the workspace ready to accept them without further root changes.

# Scope

- Root `package.json`, `pnpm-workspace.yaml`, shared TS/ESLint/Prettier/Vitest configuration.
- `LICENSE`, `.gitignore`, `.editorconfig`, `.nvmrc`, minimal `README.md` skeleton.
- Empty `packages/`, `examples/`, `e2e/` directories wired into the workspace globs.

# Detailed Requirements

1. Root `package.json`: `"private": true`, `"engines": {"node": ">=20"}`, and a `"packageManager"` field pinned to an **exact** pnpm version — determine it at implementation time with `npm view pnpm dist-tags.latest` and pin that value (ADR-002 fixes the tool, not the point version; never leave a range or placeholder). Scripts:
   - `build` / `test` / `lint` / `typecheck` = `pnpm -r --if-present run <same name>`
   - `format` = `prettier . --write` · `format:check` = `prettier . --check` (root-level, non-recursive)
2. `pnpm-workspace.yaml` with `packages: ["packages/*", "examples/*", "e2e"]`.
3. `tsconfig.base.json`: `strict: true`, `noUncheckedIndexedAccess: true`, `exactOptionalPropertyTypes: true`, `module: "NodeNext"`, `moduleResolution: "NodeNext"`, `target: "ES2022"`, `isolatedModules: true`, `forceConsistentCasingInFileNames: true`, `skipLibCheck: true`. Packages will extend this file.
4. ESLint flat config (`eslint.config.mjs`): `typescript-eslint` recommended-type-checked applied to `packages/**` using `parserOptions.projectService: true` (packages bring their own `tsconfig.json`), `eslint-config-prettier` last. Add one clearly-marked section where later issues (13/19/24/27) register package-specific rules.
5. Prettier config (`.prettierrc.json`): default options; explicit `"proseWrap": "preserve"`.
6. Vitest: root `vitest.workspace.ts` referencing package-level `vitest.config.ts` files (glob `packages/*/vitest.config.ts` and `e2e` excluded).
7. `LICENSE`: MIT, copyright holder `datastory-kit contributors`, year 2026.
8. `.gitignore`: `node_modules/`, `dist/`, `coverage/`, `*.tsbuildinfo`, `.DS_Store`, `playwright-report/`, `test-results/`.
9. `.editorconfig`: UTF-8, LF, final newline, 2-space indent.
10. `.nvmrc`: `20`.
11. `README.md`: project one-liner in English (keep the existing Japanese one-liner as a second line), section stubs: What is this / Status / Quick start (placeholder) / License. Full README lands in issue 45.
12. Do **not** add Turbo, Nx, changesets, husky, or lint-staged.

# Acceptance Criteria

- [ ] Fresh clone + `corepack enable && pnpm install` succeeds on Node 20 with no warnings about workspace config.
- [ ] `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build`, `pnpm format:check` all exit 0 on the empty workspace (`--if-present` makes recursive scripts no-ops).
- [ ] `pnpm-lock.yaml` is committed; `packageManager` pins an exact version.
- [ ] LICENSE is MIT and referenced from root `package.json` (`"license": "MIT"`).
- [ ] Top-level `packages/`, `examples/`, `e2e/` exist and are matched by the workspace globs; feature-package and example subdirectories are deliberately absent (they arrive in issues 04/12/19/24/40).

# Validation

Run exactly: `pnpm lint && pnpm typecheck && pnpm test && pnpm build && pnpm format:check` on Node 20 and Node 22. Confirm `git status` is clean after `pnpm install` (no mutated lockfile). CI enforcement arrives in issue 02.

# Dependencies

None (first issue).

# Non-goals

- No feature package code (issues 04/12/19/24).
- No CI workflows (issue 02), no security configs (issue 03).
- No CONTRIBUTING/SECURITY docs (issues 43/45).

# Design References

- DESIGN.md §4.1 (repository layout), §15 (budgets referenced by later issues)
- ADR-002 (TypeScript + pnpm monorepo)
