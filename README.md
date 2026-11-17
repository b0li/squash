# Squash — Convert, Resize & Crop Images in Your Browser

A tiny, single-file web app that lets you **convert, resize and crop images** by drag & drop — **WebP, AVIF, JPEG and PNG** — entirely in your browser. No upload, no server, no tracking: every file stays on your machine.

**Live:** [b0li.github.io/squash](https://b0li.github.io/squash/)

> **Why?** Most online image converters upload your files to some server and bury the tool under ads. Squash does the whole job client-side with the Canvas API (plus a WebAssembly encoder for AVIF), so it is instant, private, and works offline once loaded.

---

## Features

- **Drag & drop** or click to select — multiple files at once
- **Output format** — WebP, JPEG, PNG (native `canvas.toBlob`) and **AVIF** (via `@jsquash/avif` WASM)
- **Quality slider** for lossy formats (auto-disabled for lossless PNG)
- **Resize by width** — downscale only, never upscales
- **Crop** — per image, with an interactive crop box, aspect-ratio presets (Free · Original · 1:1 · 4:3 · 3:2 · 16:9) and an optional **fixed output size** in pixels
- **Rotate** — 90° steps, left or right
- **Per-image overrides** — change format, width, crop and rotation on each file individually, not just globally
- **SEO-friendly filenames** — optional slugify (`solar power system.jpg` → `solar-power-system.webp`): lowercase, umlaut expansion, spaces/dots → hyphens
- **Live size comparison** — original → result, with the % saved and the resolution per file
- **Batch download as ZIP** — with a filename you choose
- **100% client-side** — files never leave the browser; nothing is uploaded

### Supported input formats

JPG · PNG · GIF · AVIF · BMP · HEIC · HEIF · TIFF · SVG · ICO · WebP
(HEIC/HEIF are decoded via [`heic2any`](https://github.com/alexcorvi/heic2any).)

---

## Usage

1. Open [b0li.github.io/squash](https://b0li.github.io/squash/).
2. Drop one or more images onto the drop zone.
3. Pick the output **format**, **quality** and optional **width** — globally or per file.
4. Optionally **crop** or **rotate** individual images.
5. **Download** each file, or grab everything as a **ZIP**.

Every transform re-encodes from the original, so switching format or width after cropping/rotating never stacks quality loss.

---

## Run locally

It is a single `index.html` with no build step — just open it:

```bash
git clone https://github.com/b0li/squash.git
cd squash
open index.html          # or serve it: python3 -m http.server
```

> AVIF encoding loads the WASM encoder from a CDN on first use, so AVIF needs an internet connection; everything else works fully offline.

---

## Tech

- Vanilla JS, single `index.html`, no framework, no build step
- Canvas API for WebP/JPEG/PNG encoding
- [`@jsquash/avif`](https://github.com/jamsinclair/jSquash) (libavif / WebAssembly) for AVIF encoding
- [`heic2any`](https://github.com/alexcorvi/heic2any) for HEIC/HEIF decoding
- [`JSZip`](https://stuk.github.io/jszip/) for the batch ZIP download
- Hosted on GitHub Pages

## License

[GPLv2](LICENSE) © Markus Bäck · [lake-studio.de](https://www.lake-studio.de)
