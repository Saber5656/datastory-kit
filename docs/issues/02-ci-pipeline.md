# Title

CI pipeline: lint, typecheck, test, build

# Summary

Add the GitHub Actions CI workflow that gates every PR: install with frozen lockfile, lint, typecheck, unit tests (Node 20 + 22 matrix), and workspace build. Actions are pinned to commit SHAs with minimal permissions.

# Context

DESIGN.md §14 defines the CI-tested quality bar and §13.7 requires pinned actions and least-privilege workflows from the first workflow onward. This pipeline must be cheap enough to run on every PR and strict enough that later issues can rely on "CI green = contracts hold".

# Scope

- `.github/workflows/ci.yml` (CodeQL/Dependabot/audit are issue 03).
- One README line: the CI status badge.

# Detailed Requirements

1. Triggers: `pull_request` (all branches) and `push` to `main`.
2. Top-level `permissions: contents: read`. No job may request more.
3. Jobs:
   - `lint`: checkout → setup → `pnpm lint` and `pnpm format:check` (script defined in issue 01).
   - `typecheck`: `pnpm typecheck`.
   - `test`: matrix `node-version: [20, 22]` → `pnpm test -- --run` (non-watch; coverage configuration belongs to package issues).
   - `build`: `pnpm build`; uploads `packages/*/dist` via `actions/upload-artifact` with `if-no-files-found: ignore` (retention 7 days) — the workspace has no dist outputs until feature packages land.
4. Shared setup steps (use a composite action or repeat inline): `actions/checkout`, `pnpm/action-setup` (version read from `packageManager`), `actions/setup-node` with `cache: pnpm`, then `pnpm install --frozen-lockfile`.
5. Every third-party action pinned to a full commit SHA with a trailing `# vX.Y.Z` comment.
6. `concurrency: { group: ci-${{ github.ref }}, cancel-in-progress: true }`.
7. Add a status badge to README.

# Acceptance Criteria

- [ ] A PR that violates ESLint/Prettier/tsc/tests/build fails the corresponding job (verify each with a deliberate scratch commit on a test branch, then drop it).
- [ ] All actions referenced by SHA; `grep -E "uses:.*@(v[0-9]|main|master)" .github/workflows/ci.yml` returns nothing.
- [ ] Workflow has no `write` permission anywhere.
- [ ] CI completes in under 5 minutes on the empty-ish workspace.

# Validation

Open a draft PR touching a workspace file; confirm all four jobs run and pass; confirm cancellation works by pushing twice quickly.

# Dependencies

- 01 (workspace scripts exist)

# Non-goals

- CodeQL, Dependabot, `pnpm audit` gate (issue 03).
- E2E/Playwright job (issue 42 extends this workflow).
- Bundle-size budget check — issue 24 adds a `size` job to this workflow once the player exists; this issue only keeps the job layout extension-friendly (no placeholder job needed).
- Release/publish workflow (issue 46).

# Design References

- DESIGN.md §14 (testing strategy), §13.7 (supply chain: pinning, permissions)
