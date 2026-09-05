# Title

Deployment guide, README overhaul, live demo

# Summary

Write `docs/guide/deploying.md` (static hosting recipes incl. GitHub Pages), overhaul the README into the OSS front door, add CONTRIBUTING.md and CODE_OF_CONDUCT.md, and stand up the live demo of the sample scenario on GitHub Pages.

# Context

DESIGN §17 and §2.2 step 6: after `build`, a teacher needs a copy-paste path to a URL their class can open. The README is the project's first impression for both audiences (teachers via the demo link, contributors via architecture pointers). The demo doubles as continuous proof that a real deployment works (§3.4 U5).

# Scope

- `docs/guide/deploying.md`, `README.md` rewrite, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `.github/workflows/deploy-demo.yml`.

# Detailed Requirements

1. Deployment guide recipes (each: prerequisites, exact commands, `--base-path` guidance, verification step):
   - GitHub Pages via Actions (primary; artifact upload → `actions/deploy-pages`, SHA-pinned): exact commands `datastory build examples/school-library-case-ja -o _site/ja --base-path /<repo>/ja/` and `… -en -o _site/en --base-path /<repo>/en/`;
   - "any static host" generic recipe (Netlify/school LMS/local file server via `datastory preview` on a classroom machine);
   - offline classroom pattern (build on one machine, serve over LAN with `preview --host` + its §10.5 warning, or USB + local preview).
   - Troubleshooting table: wasm MIME errors (U1), blank page at wrong base path (U5), storage-unavailable banner meaning (U2).
2. README overhaul: hero paragraph (bilingual one-liner: en primary, ja subtitle), demo links (ja + en), screenshot at `docs/assets/demo-screenshot.png` (captured from the deployed ja sample after the chapter-1 smoke), "for teachers/creators" quick start (5 commands), "for contributors" map (packages table from §4.1, link to DESIGN/ISSUE_PLAN/ADRs), status badges (CI, license), link to `SECURITY.md` (complete — issue 43 is a dependency), license section.
3. CONTRIBUTING.md: dev setup (corepack/pnpm, commands), monorepo map, test tiers (§14), PR expectations (lint/typecheck/tests green, dependency-addition review note per §13.7 — this is the canonical home of that note, pointed to by SECURITY.md), issue-plan pointer ("pick an issue from docs/issues/").
4. CODE_OF_CONDUCT.md: Contributor Covenant v2.1. **Blocked-on-maintainer input**: the enforcement contact channel must be provided by the maintainer before merge (do not invent one; a `TODO(maintainer-contact)` placeholder plus a PR checklist item is the mechanism — see also the open user-decision item in the PR description).
5. Demo workflow `deploy-demo.yml`: trigger `push` to `main` with paths `packages/**`, `examples/**`, `pnpm-lock.yaml`, `.github/workflows/deploy-demo.yml` (schema/compiler/cli all affect sample builds — include the whole `packages/**`); builds both samples with the exact commands from recipe 1 plus a tiny hand-written index page linking both locales; permissions: `permissions: {}` at workflow top level, `contents: read` on the build job, `pages: write` + `id-token: write` on the deploy job only; all actions SHA-pinned per the issue 03 policy.
6. All docs English (repo policy); README's ja subtitle line kept.

# Acceptance Criteria

- [ ] Following the GitHub Pages recipe verbatim on a scratch fork produces a working deployment (recorded in PR).
- [ ] Live demo URLs load both locales with zero console errors and pass a manual smoke playthrough of chapter 1.
- [ ] README renders correctly on GitHub (links/badges/images resolve); a newcomer can find "how to write a scenario" in ≤ 2 clicks.
- [ ] CONTRIBUTING/CODE_OF_CONDUCT present and linked from README.
- [ ] Demo workflow green, SHA-pinned, minimal permissions (reviewed against §13.7).

# Validation

Scratch-fork deployment test; link checker over README/guides (add `lychee` or equivalent to CI as a soft/nightly job); demo smoke test.

# Dependencies

- 22, 23, 42, 43 (samples proven playable and SECURITY.md complete before the public front door links to them; dependency table updated)

# Non-goals

- Custom domain/branding; docs website generator (plain Markdown v1); Japanese translations (v2).

# Design References

- DESIGN.md §17 (distribution), §2.2 step 6, §10.4/§10.5 (base path/preview), §3.4 U1/U2/U5, §13.7 (workflow hardening)
