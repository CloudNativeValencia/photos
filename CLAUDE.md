# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A static photo gallery site for Cloud Native Valencia meetup events, deployed to GitHub Pages via GitHub Actions. There is no build step, no framework, and no package.json — every page is hand-written HTML/CSS/vanilla JS. Photos themselves are **not** stored in this repo; they live in Linode Object Storage (S3-compatible) at `https://cloudnativevalencia-photos-prod.es-mad-1.linodeobjects.com/public/<album>/` and pages link to them by URL.

## Structure

- `index.html` — landing page listing all albums as cards (thumbnail, title, date/blurb). One `.album-card` block per album, newest first.
- `<album>/index.html` — one folder per event, named by year+month (`202511`, `202604`, `202607`). Each contains a self-contained gallery page: a responsive photo grid plus a built-in lightbox (prev/next/close, keyboard arrows/Escape, click-outside-to-close) implemented in inline vanilla JS at the bottom of the file. No external JS libraries.
- `img/` — static site assets (e.g. the CN Valencia logo used in the header).
- `photo-urls-<album>.txt` — flat list of full-size photo URLs for an album, one per line. Working data used to generate the `<a class="photo">...` blocks in that album's `index.html`; not referenced by the site at runtime.
- `photo-keys-<album>.txt` — tab-separated object storage keys for an album (used when working out prefixes/filenames before writing URLs). Also not referenced at runtime.
- `.github/workflows/pages.yml` — the only automation: on push to `main` (or manual dispatch), checks out the repo, uploads the whole repo as a Pages artifact, and deploys via `actions/deploy-pages`.

## Working with albums

Each album's gallery `<img>` tags point at a `thumbs/` subfolder in object storage (e.g. `.../202607/thumbs/1.jpg`) while the enclosing `<a>` links to the full-size original (e.g. `.../202607/1.jpg`). Both the thumb and the full image must already exist in object storage — this repo does not generate thumbnails.

To add a new album:
1. Upload full-size photos and thumbnails to the object storage bucket under a new `public/<album>/` prefix (thumbnails under `public/<album>/thumbs/`), and make them publicly readable.
2. Create `<album>/index.html` by copying an existing album page (e.g. `202607/index.html`) and updating the album name in the title/heading/aria-label, and regenerating the repeated `<a class="photo">…</a>` blocks for the new photo set (URLs to full image + thumb, sequential alt text like `"<album> event photo 001"`).
3. Add a new `.album-card` entry to the top of `index.html`'s `.albums` section (newest album first) with a thumbnail, title, and a one-line date/blurb.

Each album page is fully self-contained (styles + lightbox script duplicated per file) rather than sharing a common CSS/JS file — keep that pattern when editing rather than introducing a shared bundle.

## Testing changes

There is no build or test tooling. Verify changes by opening the HTML files directly in a browser (or serving the repo root with any static file server) and checking that the grid renders, thumbnails load from object storage, and the lightbox opens/navigates/closes correctly.

## Deployment

Pushing to `main` automatically deploys to GitHub Pages via `.github/workflows/pages.yml`. There is no staging environment or preview step — commits to `main` go live directly.
