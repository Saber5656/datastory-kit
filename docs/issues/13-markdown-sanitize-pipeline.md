# Title

Markdown rendering and sanitization pipeline

# Summary

Implement the compile-time Markdown→sanitized-HTML pipeline used for every prose field (chapter bodies, documents, dialogue replies, prompts, explanations, hints): CommonMark+GFM tables, ruby support for furigana, and the strict allowlist + link policy of DESIGN §13.2.

# Context

This is the XSS choke point for trust boundary 2 (author content → learner browser). The player renders these strings via one `SafeHtml` component without re-sanitizing (DESIGN §13.2), so *nothing* unsafe may survive this stage. It also carries a pedagogical feature: `<ruby>` furigana, required for the elementary-school reading level (DESIGN §16.3).

# Scope

- `src/markdown.ts` in `packages/compiler`: `renderProse(md: string, opts: {allowExternalLinks: boolean, file: string}): {html: string, diagnostics: Diagnostic[]}`.
- Unit tests incl. the XSS corpus seed (full corpus grows in issue 43).

# Detailed Requirements

1. Pipeline exactly per DESIGN §13.2: `remark-parse` → `remark-gfm` → **raw-HTML gate** → `remark-rehype` (`allowDangerousHtml: true`) → `rehype-raw` → `rehype-sanitize` (schema below) → link-policy transform → `rehype-stringify`. The raw-HTML gate is an mdast transform: `html` nodes pass only when they match `/^<\/?(ruby|rt|rp)>$/` (bare tags, no attributes); all other `html` nodes are replaced with literal-text nodes of their source (rendered as visible escaped text) while emitting DS5001 (suspicious per requirement 5) or DS5103 (benign). Consequently `rehype-raw` can only materialize ruby, and the sanitize schema governs markdown-generated elements as defense-in-depth. GFM autolink literals become ordinary `<a>` nodes and flow through the same link policy (no attempt to disable them at parse level).
2. Sanitize schema (DESIGN §13.2): tags `p h1 h2 h3 h4 ul ol li table thead tbody tr th td em strong del code pre blockquote hr br a ruby rt rp`; attributes: `a[href]` only; strip all `class`/`style`/`id`/event handlers; no `img`, `svg`, `iframe`, `script`, `details`, `input`.
3. Link-policy transform — runs **after** sanitization (order matters: the sanitize schema stays minimal; the transform adds the fixed attributes):
   - `allowExternalLinks: false` (default): every `<a>` unwrapped to its text; emit warning DS5101 per link with the href in the message.
   - `true`: allow `https:` and `mailto:` only, adding `target="_blank" rel="noopener noreferrer"`; any other protocol (`http:`, `javascript:`, `data:`, `vbscript:`, protocol-relative `//`, relative paths) unwrapped + DS5102 warning.
4. Heading demotion: `h1` in prose demoted to `h2` (the page h1 belongs to the player shell); demotion emits no diagnostic.
5. Source-detection diagnostics (DESIGN §13.2): DS5001 (error) when the raw Markdown *source* contains — after lowercasing, whitespace stripping inside tokens, and HTML-entity decoding — any of: `<script`, an `on[a-z]+=` attribute pattern, or a `javascript:`/`data:`/`vbscript:` URI. DS5103 (warning) for any other raw-HTML construct the sanitizer stripped (mistyped ruby, pasted rich text). Fail loudly rather than silently strip.
6. Output HTML is deterministic (no random ids); `&` etc. entity-encoded by the stringifier.
7. Two exported functions in `packages/compiler/src/markdown.ts`:
   - `renderProse(md, opts: {allowExternalLinks: boolean, file: string}): {html, diagnostics}` — the single rendering primitive;
   - `renderLoadedProse(loaded: LoadedScenario): {rendered: RenderedProse, diagnostics: Diagnostic[]}` — maps every prose field: chapter `bodyMd → bodyHtml`; document `bodyMd → bodyHtml`; topic `reply → replyHtml`; question/final `prompt → promptHtml`, `explanation → explanationHtml`, `hints[i] → hints[i].html`. `allowExternalLinks` comes from `loaded.manifest.settings`. Compiler stages must not call remark directly (ESLint `no-restricted-imports` outside `markdown.ts`).
8. Test fixtures live as a table `packages/compiler/test/fixtures/prose-cases.json`: `{name, input, allowExternalLinks, expectedHtml, expectedCodes[]}` — expected sanitized output is pinned byte-exactly per case (serializer output is deterministic, so this is stable).

# Acceptance Criteria

- [ ] GFM table and ruby fixture render correctly (`<ruby>漢字<rt>かんじ</rt></ruby>` preserved), asserted against pinned `expectedHtml`.
- [ ] XSS seed corpus all neutralized with pinned expected outputs and codes: `<script>alert(1)</script>`, `<img src=x onerror=alert(1)>`, `[x](javascript:alert(1))`, `<a href="data:text/html,...">`, `<svg onload=…>`, `<iframe>`, event-handler attributes, `<style>` (each row states whether stripped content survives as text or disappears).
- [ ] Link policy fixtures: both modes produce the specified DS510x warnings and pinned outputs; allowed external links carry both `target="_blank"` and `rel="noopener noreferrer"`; a GFM autolink literal follows the same policy.
- [ ] DS5001 fires on `<script` input even though output is clean; DS5103 fires on `<b>bold</b>` and the output shows `&lt;b&gt;bold&lt;/b&gt;` as visible text; a raw `<table>` in source is escaped to text (does **not** survive as an element) while a GFM pipe table renders as `<table>` (the gate/allowlist distinction, asserted in one fixture pair).
- [ ] `renderLoadedProse` maps every prose field of the `valid-full` fixture (spot-assert one of each kind).
- [ ] Two consecutive runs are byte-identical for the full valid fixture (determinism).

# Validation

Unit tests per above; snapshot of `valid-full` fixture prose outputs. Issue 43 extends the corpus and re-runs it against the *built sample bundle*.

# Dependencies

- 12, 18 (diagnostic emission conventions)

# Non-goals

- Player-side rendering component (`SafeHtml`, issue 24/28) — this issue is compile-time only.
- Inline images in prose (explicit non-goal, DESIGN §3.2).
- Syntax highlighting of code blocks (v2 nicety).

# Design References

- DESIGN.md §13.2 (sanitization policy), §5.4 (prose capabilities), §11.1 (DS5xxx), §16.3 item 4 (furigana)
