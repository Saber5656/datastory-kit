# Title

npm release workflow with provenance

# Summary

Implement the v1 release machinery per DESIGN §17/§13.7: lockstep versioning across the four packages, a tag-triggered CI publish with npm provenance (OIDC), package-content audits, and the release checklist tying in the security gates.

# Context

Publishing is the last trust boundary (DESIGN §13.1 boundary 4): a compromised or sloppy release ships to authors' machines and school deployments. The workflow must make the secure path the only path — publish only from CI, only from tags, only with provenance, never from a laptop.

# Scope

- `.github/workflows/release.yml`, `scripts/release/` helpers (version sync, pack audit), package publish metadata polish, `docs/guide/releasing.md` (maintainer runbook).

# Detailed Requirements

1. Versioning: lockstep — one `version` across `@datastory/{schema,compiler,player,cli}`; helper `scripts/release/set-version.mjs <semver>` updates all four + inter-package dependency ranges (`workspace:*` → exact on publish via pnpm). `formatVersion` (§5.3) is explicitly *not* touched by releases (design-governed).
2. Trigger: push of tag `v*.*.*` matching the manifest versions (workflow verifies match, else fails). Manual `workflow_dispatch` defines input `dry_run` (boolean, required, must be `true` — the workflow fails fast if a manual dispatch attempts a real publish).
3. Release job sequence: frozen install → full CI suite reuse (lint/typecheck/test/build; E2E Chromium tier) → pack audit → publish step gated by `environment: release` → `pnpm publish -r --access public --provenance --no-git-checks` via OIDC (`id-token: write`; **no** npm token secret — npm Trusted Publishing). Maintainer manual preconditions documented in the runbook and checklisted: npm Trusted Publishing configured for all four packages **and 2FA enforced on the npm org** (§13.7).
4. Pack audit script: `pnpm -r pack` then assert per package — `files` allowlist honored (no tests/fixtures/src leaks), no `postinstall`/lifecycle scripts (§13.7), LICENSE + README present in each tarball, `private` absent/false, `exports`/`types` resolve, `bin` resolves for cli, player tarball contains `dist/` with the wasm asset, total tarball sizes within budget (player ≤ 3 MB). Per-package publish metadata requirements (exact): every `package.json` has `license: "MIT"`, `repository`, `files`, `exports` with `types` condition; cli additionally `bin`; player additionally ships `dist/` only.
5. GitHub Release: auto-created from the tag with generated notes (Conventional-ish commit grouping best-effort) + attached built sample bundle zip (ja) as a convenience artifact.
6. Runbook `docs/guide/releasing.md`: preconditions (security checklist from issue 43 green, ISSUE_PLAN completion state), commands (`set-version` → PR → merge → tag), rollback guidance (`npm deprecate`, never unpublish >72h), provenance verification snippet (`npm audit signatures`).
7. Workflow hardening: SHA-pinned actions, job-scoped minimal permissions, environment `release` with required reviewers (maintainer approval gate before publish step).

# Acceptance Criteria

- [ ] Dry-run via `workflow_dispatch` (`dry_run: true`) completes: version-match check, full test tier, pack audit — all green, publish step skipped with clear log; `dry_run: false` dispatch fails fast.
- [ ] Pack audit demonstrably fails on: a leaked `src/` file, an added `postinstall`, a missing LICENSE (scratch-verified each, reverted).
- [ ] Tag/manifest version mismatch fails fast with a clear message.
- [ ] Publish-path mechanics validated in CI against a throwaway verdaccio registry (no provenance there — mechanics only); **provenance itself** is verified on the first real npm publish with `npm audit signatures` output recorded in the release issue.
- [ ] Publish job carries `environment: release` in YAML; runbook + release checklist include the maintainer item "GitHub `release` environment has required reviewers enabled" (environment settings are UI-side and cannot be proven from YAML alone).
- [ ] Runbook walkthrough by a second maintainer/agent without assistance; npm-org-2FA + Trusted-Publishing preconditions checklisted.

# Validation

Dry-run + scratch-sabotage tests above; verdaccio-based integration publish in CI if feasible (document decision either way).

# Dependencies

- 02, 03, 17, 22, 24, 43 (all packages publishable; security checklist is a release precondition — dependency table updated)

# Non-goals

- Changesets/independent versioning (lockstep chosen, §17); prerelease channels (add when needed); marketing/announcement automation.

# Design References

- DESIGN.md §17 (release), §13.7 (supply chain), §13.1 boundary 4
