# Title

Image evidence viewer with zoom and pan

# Summary

Implement the image half of the Evidence view per DESIGN §9.3: thumbnail grid of visible images and a lightbox with zoom (1×–4×), pan, captions, and full keyboard/screen-reader support.

# Context

Images carry spatial clues (floor plans, photographed details); zoom is functional, not decorative — learners must be able to inspect details on classroom projectors and small tablets alike. Alt text is authored and mandatory (issue 06), so the a11y path is data-complete.

# Scope

- `packages/player/src/views/ImageGrid.tsx`, `packages/player/src/views/ImageLightbox.tsx`, `packages/player/src/hooks/useZoomPan.ts` + CSS Modules; component and hook tests alongside. No third-party zoom/lightbox dependency.

# Detailed Requirements

1. Grid: images from issue 25's `visibleImages()` selector in bundle order (§6.2 — never locally recompute unlock rules or re-sort), thumbnails (CSS `object-fit: cover`, fixed aspect tile) with title below; unread = id ∉ `facts.readEvidence`; clicking sets `activeView {kind:"image", id}`, opens the lightbox, and fires `markEvidenceRead(id)` (idempotent).
2. Lightbox: dialog-pattern overlay (focus trap, `Esc` closes, focus returns to the invoking tile). Content: the image (native resolution, `alt` from bundle), title, caption — `title`/`caption`/`alt` render only as React text nodes/attributes (plain strings per schema; no `SafeHtml`, no `dangerouslySetInnerHTML`); `src` is the compiled relative `media/...` path used as-is (no URL construction, no external fetches — DESIGN §13.6). Zoom controls per below.
3. Zoom/pan: buttons `+`/`−`/reset and keyboard `+`/`-`/`0`; wheel zoom toward cursor; drag pan when zoomed (pointer events; touch pinch best-effort via two-pointer distance). Scale clamps 1×–4×; pan clamps to image bounds; double-click toggles 1×↔2×.
4. Transform math isolated in a pure hook `useZoomPan` (unit-testable without DOM).
5. Loading/failure: spinner until `load`; on `error` show a fallback panel containing the localized `evidence.imageError` message plus the image `title` and `alt` text (content remains accessible with a broken asset); no retry loop; the item is not marked loaded.
6. Arrow-key prev/next image within the lightbox (visible set order), announced via the dialog title change.
7. All strings via i18n; keys in both catalogs.

# Acceptance Criteria

- [ ] Grid shows only visible images; unread dots behave as in issue 30.
- [ ] Keyboard-only session: open tile → zoom with `+` → pan with arrows (when zoomed, arrows pan; when 1×, arrows switch image — document and test both) → `Esc` returns focus to tile.
- [ ] `useZoomPan` unit tests: clamping, cursor-anchored wheel zoom math, double-click toggle.
- [ ] Broken-src fixture renders the fallback with `evidence.imageError` + title + alt (asserted), no retry loop.
- [ ] axe smoke passes (dialog semantics, alt present); ja/en render with zero missing-key throws — required keys include `evidence.imageError`, zoom in/out/reset, close, previous/next, loading.

# Validation

Component + hook unit tests; manual touch check on a tablet (or DevTools touch emulation) noted in PR.

# Dependencies

- 28

# Non-goals

- Image annotations/markers (v2); deep-zoom tiling for huge images (assets capped at 2 MiB, §5.5); gallery slideshow autoplay.

# Design References

- DESIGN.md §9.3 (image viewer), §5.5 (image constraints), §9.1 (dialog/focus contract)
