# Biometric Vision — Honest rPPG Demo

A single-file browser demo that estimates your **pulse** and **respiration rate** from a plain webcam feed, using remote photoplethysmography (rPPG). No wearables, no sensors — just the subtle color and brightness changes in your skin as blood pumps and your chest moves.

The app is intentionally "honest": it labels its own confidence (`collecting` / `unstable` / `usable`), shows you the raw signals it's working from, and every number on screen is clickable for a plain-English explanation of what it means and how it was computed.

**Not for clinical use.** This is an educational toy to make the math behind rPPG tangible.

## What it does

- Captures webcam video in the browser (never leaves your machine).
- Downsamples each frame to 160×120 and averages pixel values inside five facial ROIs (forehead, nose, under-eye, left/right cheek).
- Derives a chromatic pulse signal (`green − (red + blue)/2`) and a luma-based respiration signal.
- Runs a bandpass filter + frequency analysis to estimate BPM (0.7–3.5 Hz) and breaths per minute (0.1–0.6 Hz).
- Renders live waveforms, a frequency bar chart, and ROI overlays.
- Every metric, waveform, bar, and ROI is clickable — opens a live card explaining the definition and current computation.

## Running it

Open `index.html` in a browser, or serve the directory:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000`. `getUserMedia` requires a secure context, so `file://` won't work — use `localhost` or HTTPS.

## Stack

Pure vanilla JS, HTML, and CSS in a single file. No build system, no package manager, no dependencies. The "one HTML file you can open anywhere" property is the product.

See `CLAUDE.md` for architecture notes.

## License

MIT — see [LICENSE](LICENSE).

## More

Built by Binmucker. See more of what I build at [Binmucker.com](https://Binmucker.com).
