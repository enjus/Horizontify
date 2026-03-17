# Horizontify

Convert portrait photos to landscape — no upload, no account, no install.

Horizontify pads portrait (or too-narrow) photos into **4:3** or **16:9** by filling the side bars with a blurred, darkened version of the original image. Everything runs in the browser via the Canvas API. Nothing leaves your device.

## Use it

Open `index.html` in any modern browser. That's it.

## What it does

Drop or paste a photo. Horizontify will:

- **Pillarbox** it to 4:3 or 16:9, with blurred bars derived from the image itself
- Offer **four crop alternatives** (Smart, Top, Center, Bottom) if you'd rather trim than pad
- **Smart crop** uses heuristics (eyes ~25% from top, 30% headroom) to guess the best vertical slice — upgraded to face detection on Chrome for Android/ChromeOS when the `FaceDetector` API is available
- Let you **toggle between 4:3 and 16:9** at any time and re-download
- If the photo is already wide enough for the chosen ratio, show a notice and offer the original for download

All output is a high-quality JPEG (0.92 quality, capped at 4096px wide).

## No build step

One file. No npm, no bundler, no server required. For live-reload during development:

```sh
npx serve .
# or
python -m http.server 8080
```

## Browser support

Any modern browser with Canvas API support. Face detection (Smart crop upgrade) requires Chrome on Android or ChromeOS.
