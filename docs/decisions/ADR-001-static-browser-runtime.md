# ADR-001: Ship the player as a fully static, browser-only runtime

- Status: Accepted
- Date: 2026-07-10
- Deciders: Product owner (user decision, 2026-07-10 requirements interview) + design agent

## Context

datastory-kit produces narrative investigation learning materials ("SQL Murder Mystery"-style) aimed primarily at school classrooms. Deployment targets are teachers and educational content creators who typically cannot run servers, and school networks that are often restrictive or offline. Learner privacy is a hard requirement: materials may be played by minors.

## Decision

The compiled material is a **fully static web site**. Everything — story content, evidence, the SQL engine (WebAssembly SQLite), answer judging, and progress tracking — runs client-side in the browser. The build output can be hosted on any static file host (GitHub Pages, school LMS static storage, a USB stick opened via a local static server).

Concretely:

1. `datastory build` emits a self-contained directory of static files (HTML/JS/CSS/WASM/JSON/SQLite/images).
2. The player makes **no network requests** other than same-origin fetches of its own build artifacts.
3. Answer judging happens client-side against salted hashes (see ADR-007); no answer server exists.
4. Learner progress is stored in `localStorage` only (see DESIGN.md §7.4).
5. No telemetry, no analytics, no third-party resources (fonts, CDNs) in the built output.

## Consequences

Positive:

- Zero hosting cost and trivial deployment for teachers.
- Works offline after first load; suitable for restricted school networks.
- Strongest possible privacy posture: no learner data leaves the device.
- Security surface shrinks to content sanitization + supply chain (no server-side attack surface).

Negative:

- No teacher dashboard, cross-device sync, or class analytics in v1 (deferred to v2).
- Answers embedded in the bundle can be brute-forced by a determined learner; hashing is casual spoiler protection only (ADR-007).
- Progress is lost if the browser profile is cleared; an export/import feature is a v2 candidate.
- Strict CSP must be designed in from the start because there is no server to compensate (DESIGN.md §13.3).

## Alternatives considered

- **Server-rendered web app (Node/Python)**: rejected — hosting burden on teachers, learner data handling obligations, larger attack surface.
- **Local CLI/terminal player**: rejected — poor narrative presentation, excludes non-engineer learners (user explicitly chose browser static in the requirements interview).
- **Distribute raw data + docs only**: rejected — does not deliver the guided, gated play experience that defines the product.
