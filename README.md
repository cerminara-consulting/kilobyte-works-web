# Kilobyte Works — Web

The canonical mechanical spec lives here.

This repo is **just** the playable web build — a single self-contained HTML file (no build, no deps, no network). Use it as the reference implementation. The iOS port tracks the spec verbatim.

- **Canonical file:** [`kilobyte-works.html`](./kilobyte-works.html) — 44.7 KB, drop into any browser.
- **Spec:** [`SPEC.md`](./SPEC.md) — full mechanical spec including narrative arcs, endings, and countermeasure costs.
- **iOS port:** [`cerminara-consulting/kilobyteworks`](https://github.com/cerminara-consulting/kilobyteworks)

## Run

Open `kilobyte-works.html` directly in a browser, or `file://` it. The game uses `localStorage` for autosave so:

- It works offline after first load.
- Each browser keeps its own save — clearing site data wipes the run.

## Deploy

Drop the file on any static host. The file is self-contained — no MIME-type wrangling, no asset pipeline. Cloudflare Pages, GitHub Pages, Vercel static, `python -m http.server`, S3, or `lpr` to a real Macintosh (worst case).

## Spec parity

The iOS SwiftUI port in `cerminara-consulting/kilobyteworks` mirrors this spec's mechanics (cost ladder, prestige math, narrative triggers, endings). When the two disagree, **this repo wins** — the web is the canonical reference until the iOS port catches up at v1.0.
