# Morph

A single-file, browser-only file converter and toolkit. No server, no upload, no accounts — every conversion, edit, and tool runs on your device using WebAssembly and native browser APIs. Open `morrph.netlify.app` in any modern browser and it just works.

---

## Contents

- [Quick start](#quick-start)
- [Features](#features)
  - [Convert](#convert)
  - [Tools](#tools)
  - [Batch](#batch)
  - [History](#history)
- [Supported formats](#supported-formats)
- [How it works](#how-it-works)
- [Known limitations](#known-limitations)
- [Tech stack](#tech-stack)
- [Privacy](#privacy)
- [Browser support](#browser-support)
- [File structure](#file-structure)
- [Troubleshooting](#troubleshooting)

---

## Quick start

1. Download `morph.html`.
2. Open it in Chrome, Edge, Firefox, or Safari (double-click, or drag it into a browser tab).
3. That's it — no install, no build step, no dependencies to set up locally. Everything it needs loads from a CDN on first use.

You can also drop it in any static file host (GitHub Pages, Netlify, S3, etc.) if you want a shareable URL — it's a single HTML file with no backend.

---

## Features

### Convert
The main two-box flow: drop a file in, it's auto-detected, pick a target format from a dropdown scoped to what's actually valid for that file type, hit **Convert**, get a live preview, download.

- Full document matrix (see table below)
- Full image matrix (JPG, PNG, GIF, WEBP, SVG — all to all)
- Audio matrix (MP3, WAV, M4A) with bitrate control and trim
- Video (MP4, MOV) with resolution, FPS, quality/compression, and trim
- Audio extraction from video (MP4/MOV → MP3/WAV/M4A)
- Video → GIF

When the uploaded file is an image, an **Image tools** panel appears with:
- **Resize** — numeric width/height, with an aspect-ratio lock
- **Crop** — drag-to-resize crop rectangle right on the image
- **Compress** — quality slider with a live before/after size comparison
- **Remove Background** — on-device ML segmentation, outputs a transparent PNG
- **OCR** — extract text from an image, copy or download as `.txt`

### Tools
Dedicated multi-file utilities, each its own focused screen:

| Tool | What it does |
|---|---|
| PDF Merge | Combine multiple PDFs into one, reorderable before merging |
| PDF Split | Split into individual pages (zipped) or extract a page range |
| PDF Compress | Optimize a PDF's internal structure to shrink file size |
| PDF Password | Lock a PDF behind a password |
| Watermark PDF | Stamp text diagonally across every page |
| Images → PDF | Combine multiple images into one PDF, in order |
| PDF → Images | Export every page as a JPG or PNG (zipped if multi-page) |
| Merge Audio | Stitch multiple audio clips into one file |

### Batch
Drop any number of mixed files at once. Each gets its own row with an auto-detected source format and an editable target dropdown. Convert them one at a time or all at once, then **Download all as ZIP**.

### History
Every conversion and tool action logs to a local table — file name, action, size, timestamp. Persists across browser sessions using `localStorage`. Only metadata is stored, never the files themselves. Clearable at any time.

A dark/light theme toggle lives in the top-right corner and remembers your choice.

---

## Supported formats

### Documents
| From | Converts to |
|---|---|
| PDF | DOCX, TXT, MD, ODT, HTML, CSV, JPG, PNG |
| DOCX | PDF, TXT, MD, ODT, HTML |
| DOC *(legacy)* | Detected, but not convertible — see [limitations](#known-limitations) |
| TXT | PDF, DOCX, MD, HTML |
| ODT | PDF, DOCX, TXT, MD |
| MD | PDF, DOCX, HTML, TXT |
| CSV | XLSX, JSON, XML, PDF |
| XLSX | CSV, PDF, JSON, XML |
| XML | JSON, CSV |
| JSON | XML, CSV |

### Images
JPG, JPEG, PNG, GIF, WEBP, SVG — every format converts to every other format.

### Audio
MP3, WAV, M4A — every format converts to every other format, with bitrate control and trimming.

### Video
MP4, MOV — convert between the two, extract audio to MP3/WAV/M4A, or export to GIF. Resolution, FPS, quality, and trim are all adjustable.

---

## How it works

Morph loads a handful of well-established open-source libraries from a CDN, all running entirely client-side:

- **Documents** — `pdf.js` (reading PDFs), `jsPDF` (writing PDFs), `mammoth.js` (reading DOCX), `SheetJS` (XLSX/CSV), a hand-built minimal DOCX/ODT writer, `marked` + `turndown` (Markdown ↔ HTML)
- **Images** — the Canvas API for all raster conversions, resizing, cropping, and compression
- **Audio/Video/GIF** — `ffmpeg.wasm`, a full build of FFmpeg compiled to WebAssembly, loaded on first use (~30MB, then cached for the session)
- **Background removal** — `@imgly/background-removal`, an on-device segmentation model (~40MB, loaded on first use)
- **OCR** — `Tesseract.js`, an on-device text-recognition engine (loaded on first use)
- **PDF tools** — `pdf-lib` for merge/split/compress/watermark

Heavy engines (FFmpeg, background removal, OCR) are only fetched the moment you actually use that feature — the app doesn't download them upfront.

---

## Known limitations

Being upfront about what this can and can't do:

- **Document fidelity** — Text, paragraphs, and simple tables carry over cleanly. Complex layouts, custom styling, embedded images inside a source document, headers/footers, and footnotes are not preserved. This is a fundamental tradeoff of doing conversion entirely in-browser rather than a limitation to be fixed later.
- **Legacy `.doc` files** — The old binary Word format (pre-2007) can't be parsed by any JavaScript library running in a browser. Morph detects `.doc` files but will show a clear error asking you to re-save as `.docx` in Word first.
- **Raster → SVG** — This embeds the original image inside an SVG wrapper; it does not trace it into actual vector paths. True vectorization needs different, more specialized tools.
- **PDF Compress** — Uses structural optimization (removing redundant objects), which is safe and preserves quality, but won't shrink an already-optimized or scanned PDF by much. It intentionally does not re-rasterize pages to force smaller sizes, since that would degrade text quality and searchability.
- **PDF Password Protection** — Uses a community-maintained fork of the standard PDF library, since the mainstream library can't write encrypted PDFs. It's functional but less battle-tested than the rest of the toolkit — if it errors, that's the most likely reason.
- **First-use downloads** — Audio/video, GIF, background removal, and OCR each load their own engine (30–80MB combined) the first time you use them. This is a one-time cost per browser session; everything is cached afterward.

---

## Tech stack

Plain HTML, CSS, and vanilla JavaScript — no framework, no build step, no bundler. Everything lives in one `.html` file for maximum portability: copy it anywhere and it runs.

---

## Privacy

Nothing is uploaded, ever. Every conversion, edit, and tool operation happens locally in your browser using WebAssembly and native browser APIs. The only thing that persists is the History log (file names, actions, sizes, timestamps) in your browser's `localStorage` — never the file contents.

---

## Browser support

Works in current versions of Chrome, Edge, Firefox, and Safari. Requires a browser with WebAssembly, Canvas, and dynamic `import()` support (all standard in any browser from the last few years). Some heavier features (FFmpeg, background removal) will be slower on older or lower-powered devices since all processing happens on your machine rather than a server.

---

## File structure

```
morph.html   ← everything: markup, styles, and logic in one file
```

There is nothing else to install or configure.

---

## Troubleshooting

- **"Legacy .doc files can't be parsed in-browser"** — expected behavior; re-save the file as `.docx` and re-upload.
- **A first conversion in Audio/Video, Background Removal, or OCR feels slow** — that's the one-time engine download; subsequent conversions in the same session are fast.
- **Password protection fails** — see the limitation above; try again, or use the Watermark/Compress tools if password locking isn't strictly required.
- **Large video files are slow or the tab feels sluggish** — all processing runs on your CPU in the browser tab; very large files will take real time and memory. There's no way around this without sending the file to a server, which Morph deliberately never does.
