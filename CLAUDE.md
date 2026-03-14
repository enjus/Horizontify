# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

**Horizontify** — a single-file, zero-dependency web app that pillarboxes portrait photos into 4:3 or 16:9 landscape by filling the bars with a blurred version of the original. All processing happens in the browser via the Canvas API; nothing is uploaded anywhere.

**Deliverable**: `index.html` only. No build step, no npm, no server. Open directly via `file://` in any modern browser.

## Running the app

Open `index.html` directly in a browser — no server needed. For local development with auto-reload, any static file server works:

```
npx serve .
# or
python -m http.server 8080
```

There are no tests, no linter config, and no build pipeline.

## Architecture

Everything lives inline in `index.html`: CSS in `<style>`, JS in `<script>`. The sections in order:

1. **Constants** — tuning values (`BLUR_RADIUS_BASE`, `BLUR_OVERSIZE_MULT`, `SMART_SUBJECT_FRAC`, etc.)
2. **State** — single `state` object: `{ originalFile, originalImage, landscapeURL, ratio, ratioLabel }`
3. **DOM refs** — captured once at module level (except download buttons, which are always fetched fresh by ID to survive `cloneNode` replacements)
4. **Init / event wiring** — `initApp()` at the bottom
5. **Canvas pipeline** — `computeCanvasDimensions → renderPillarbox`
6. **Crop alternatives** — `renderCropAlternatives → renderThumb / downloadCrop / tryFaceDetection`
7. **UI helpers** — show/hide functions, `resetToUpload`

## Canvas rendering pipeline

```
computeCanvasDimensions(imgW, imgH)
  → canvasW = min(imgW, 4096)
  → canvasH = round(canvasW × state.ratio)   // 0.75 for 4:3, 0.5625 for 16:9
  → scale photo to fit, centered

renderPillarbox(ctx, img, dims)
  1. Blurred background: draw oversized by (blurRadius × BLUR_OVERSIZE_MULT) to absorb edge bleed
  2. Dark overlay: rgba(0,0,0,0.25) to visually separate bars
  3. Original image centered, sharp
  4. 1px separator line around the centered image
```

## Crop alternatives

Four thumbnail canvases rendered below the pillarbox: **Smart**, **Top**, **Center**, **Bottom**.

- `cropH = round(W × state.ratio)`
- **Smart offset**: `round(H × SMART_SUBJECT_FRAC − cropH × SMART_CROP_EYE_FRAC)`, clamped to `[0, H−cropH]`
  - `SMART_SUBJECT_FRAC = 0.25` — portrait eyes sit ~25% from top of frame
  - `SMART_CROP_EYE_FRAC = 0.30` — place eyes at 30% from top of the crop (headroom)
- If `FaceDetector` API is available (Chrome on Android/ChromeOS), it upgrades the Smart thumbnail async and updates the label to "face detected"
- **`wireDl` pattern**: download buttons are `cloneNode`-replaced on every image load to prevent stale listeners; always call `document.getElementById(id)` inside `wireDl`, never use captured module-level refs for these buttons
- Full-resolution crops are generated on-demand in an off-screen canvas on download click (not stored in memory)

## Aspect ratio state

`state.ratio` and `state.ratioLabel` drive every dimension calculation. Switching the `[4:3] / [16:9]` toggle calls `setRatio()`, which updates state and re-runs the full render pipeline. `ratioLabel` (`'43'` or `'169'`) is embedded in download filenames.

## Landscape handling

If `img.naturalWidth >= img.naturalHeight`, the image is treated as already landscape: the pillarbox canvas is hidden, a notice badge is shown, and the original file is offered for download via `URL.createObjectURL` (stored in `state.landscapeURL`, revoked on reset).

## Key tuning constants

| Constant | Value | Purpose |
|---|---|---|
| `BLUR_RADIUS_BASE` | 24 | Base blur radius (px) for background |
| `BLUR_OVERSIZE_MULT` | 2.5 | Multiplier to oversize blurred draw and absorb edge bleed |
| `DARKEN_ALPHA` | 0.25 | Dark overlay opacity over blurred bars |
| `SEPARATOR_ALPHA` | 0.15 | Opacity of the subtle edge line around centered image |
| `JPEG_QUALITY` | 0.92 | `toDataURL` quality argument |
| `MAX_CANVAS_WIDTH` | 4096 | Caps canvas width for very large images |
| `THUMB_W` | 240 | Thumbnail canvas width (px); height = `round(240 × state.ratio)` |
| `SMART_SUBJECT_FRAC` | 0.25 | Fraction from top of frame where portrait eyes typically sit |
| `SMART_CROP_EYE_FRAC` | 0.30 | Fraction from top of crop to place the eyes (headroom) |

## Design tokens

Dark theme CSS custom properties on `:root`:
- `--bg: #0f0f11`, `--surface: #1a1a1f`, `--surface-2: #25252d`
- `--accent: #6c63ff` (purple), `--success: #3ecf8e`, `--warn: #f6ad55`, `--danger: #fc5c7d`
- Fonts: **Syne 800** (headings), **DM Sans 400/500** (body), **DM Mono 400/500** (labels, mono text) — loaded from Google Fonts
