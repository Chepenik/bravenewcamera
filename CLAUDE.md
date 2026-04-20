# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file browser demo ("Biometric Vision - Honest rPPG Demo") that estimates pulse and respiration from webcam video using remote photoplethysmography (rPPG). The entire app — HTML, CSS, and JS — lives in `index.html`. There is no build system, no package manager, no tests, and no dependencies.

## Running

Open `index.html` directly in a browser, or serve the directory (e.g. `python3 -m http.server 8000`) and navigate to it. `getUserMedia` requires a secure context, so use `http://localhost` or `https://` — opening from `file://` will fail camera access in most browsers.

## Architecture

All logic runs inside one IIFE at the bottom of `index.html` (starts ~line 512). Key pieces:

- **`CONFIG`** (~line 516) — tunable constants: analyzer canvas size (160×120 downsampled frames), target sample rate (30 Hz), pulse band 0.7–3.5 Hz, respiration band 0.1–0.6 Hz, smoothing alpha, sample buffer caps.
- **`ROIS`** (~line 541) — five facial regions (forehead, nose, under-eye, L/R cheek) as fractional rects of a face-guide box. The overlay draws these; the analyzer averages pixels inside each.
- **`state`** — single mutable store for stream, RAF loop, ring buffers (`pulse`, `resp`, `luma`, `motion`, `time`), and computed metrics.
- **Main loop** (`loop` → `sampleVideoFrame` → `appendSample` → `render`) — RAF-driven. Each frame draws the `<video>` into a hidden 160×120 canvas, sums RGB across ROIs, derives a chromatic pulse signal `g − (r+b)/2` and a luma-based respiration signal, and pushes to bounded series.
- **Metrics** (`updateMetrics`, `estimatePulse`, `estimateRespiration`, runs on `metricIntervalMs` timer) — applies `bandpassApprox` (IIR-style filter chain) and a naive frequency analysis over `frequencyBins` to produce BPM, respiration rate, and quality labels (`collecting` / `unstable` / `usable`). Quality gates UI color and whether a BPM is displayed.
- **Rendering** — overlay canvas draws the face guide + ROI boxes; two waveform canvases (`ppgCanvas`, `respCanvas`) draw filtered signals; a bar chart shows frequency bins. DPR-aware resizing via `resizeCanvasToDisplay`.

Start/stop is token-guarded (`state.startToken`) so a stop during `getUserMedia` await doesn't leave a stale stream running. `stopCapture` always tears down tracks, timers, and RAF.

## Conventions

- Pure vanilla JS — do not introduce frameworks, bundlers, or external scripts. The "single HTML file you can open anywhere" property is the product.
- DOM handles are cached once in `cacheDom()`; reference them via the `dom` object rather than re-querying.
- Tunables belong in `CONFIG` / `COLORS` / `ROIS`, not inline magic numbers.
- The UI intentionally communicates uncertainty ("honest" demo) — preserve the `collecting` / `unstable` / `usable` quality gating and the "not for clinical use" framing when editing labels or metric logic.

## Educational features

Every visible metric, ROI row, frequency bar, waveform, and overlay is clickable: clicking opens a floating card with a plain-English definition plus a live breakdown of how the current value was computed. The pattern:

- **Registry** — `EXPLAIN` (near `ROIS`, ~line 548) maps string keys to `{ title, body(extra?) }`. `body` returns an HTML string; treat it as a view function invoked on open and on every `refreshExplanation()` tick. **Bodies must read tunables from `CONFIG` / `ROIS` rather than hardcode numbers** so they stay truthful if constants change.
- **Tagging** — elements opt in via `class="explain-target"` + `data-explain="<key>"`. For dynamically-indexed targets (the frequency-scan bars), also pass `data-bar-index="<n>"`, which arrives as `extra.barIndex` in the body function.
- **Live updates** — `updateMetrics()` calls `refreshExplanation()` at the end of each tick; any card open while metrics update will re-render its body with the current `state`.
- **Hint banner** — `#explainHint` is shown once per browser (persisted under `localStorage` key `bv.explainHintSeen`) and dismissed on first click anywhere that opens an explanation.
- **Overlay clicks** — `.overlay-display` sets `pointer-events: none` so clicks pass through to the video; the CSS rule `.overlay-display.explain-target { pointer-events: auto; }` re-enables interaction only for the explain-tagged overlays.

When adding new metrics or UI elements, follow the same pattern: give them `explain-target` + a unique `data-explain` key, add a registry entry, and if their value is live-computed, read from `state` inside `body()` so the card stays accurate while open.
