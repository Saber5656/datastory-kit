# Title

Repository security baseline: Dependabot, CodeQL, audit gate, SECURITY.md skeleton

# Summary

Install the supply-chain rails required by DESIGN §13.7: Dependabot for npm and GitHub Actions, CodeQL scanning, a `pnpm audit` CI gate, and a SECURITY.md skeleton with the vulnerability-reporting policy.

# Context

The repository is public and intended for classroom-adjacent use; supply-chain hygiene is a launch requirement, not a post-launch hardening pass (DESIGN §13). This issue adds the automated rails; the full security verification suite (sanitizer corpus, CSP tests) is issue 43.

# Scope

- `.github/dependabot.yml`, `.github/workflows/codeql.yml`, an `audit` job added to `ci.yml`, `SECURITY.md`.

# Detailed Requirements

1. `dependabot.yml`: two update ecosystems — `npm` (root directory, weekly, grouped into `npm-minor-patch` for minor+patch; majors individual) and `github-actions` (weekly). Labels: `dependencies`.
2. `codeql.yml`: language `javascript-typescript`; triggers `pull_request`, `push` to `main`, and weekly `schedule`; `permissions: security-events: write, contents: read`; actions pinned by SHA.
3. `.github/workflows/ci.yml` gains an `audit` job (reusing issue 02's setup pattern): `pnpm audit --prod --audit-level=high` — fails on high/critical advisories in production dependencies. Document the override procedure (temporary `pnpm.auditConfig.ignoreCves` in root package.json with a linked issue) in SECURITY.md.
4. `SECURITY.md` skeleton sections: Supported versions ("no supported release before the first npm publish; afterwards the latest released 0.x line of the v1 design") · Reporting a vulnerability (GitHub private vulnerability reporting; 90-day coordinated disclosure) · Security model summary (link to DESIGN §13; full data-inventory table lands in issue 43) · Dependency policy (audit gate, Dependabot, "no postinstall scripts in published packages"; the PR-level dependency-budget review note is owned by CONTRIBUTING.md, issue 45 — state the pointer).
5. Add a note in SECURITY.md that GitHub repo settings (secret scanning, push protection, private vulnerability reporting) must be enabled by a maintainer — list them as a checklist. (Settings themselves are done by the repo owner, not by code.)

# Acceptance Criteria

PR-local:

- [ ] Workflow/config YAML is valid (actionlint or `gh workflow view` parse), all actions SHA-pinned, permissions minimal.
- [ ] `audit` job fails on a scratch branch after `pnpm add -w lodash@4.17.15` (known high-severity prototype-pollution advisories), then passes after reverting `package.json` + `pnpm-lock.yaml` (record both runs in the PR, drop the scratch commit).
- [ ] SECURITY.md exists with all sections above; maintainer settings checklist present.

Post-merge (maintainer records in the issue before closing):

- [ ] Dependabot shows active in Insights → Dependency graph.
- [ ] First CodeQL run on the default branch completes green.

# Validation

Scratch-branch experiment for the audit gate as specified above; observe first CodeQL and Dependabot runs post-merge. Maintainer confirms the settings checklist items are enabled in the GitHub UI.

# Dependencies

- 01, 02

# Non-goals

- npm publish provenance and release hardening (issue 46).
- Application-level security tests (issue 43).
- Branch-protection rulesets (assumed already managed by the repo owner).

# Design References

- DESIGN.md §13.7 (supply chain & release), §13.1 (boundary 4)
