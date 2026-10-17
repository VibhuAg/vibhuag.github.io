# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Single-page personal website, live at https://vibhuagrawal.dev (custom domain set by `CNAME`; GitHub Pages serves the `main` branch, and vibhuag.github.io redirects there). There is no framework or bundler: the whole site is `index.html`, styled with Tailwind CSS v4 utilities plus custom component classes in `src/input.css`. There are no tests or linters.

## Build

```sh
npm install
npm run build   # minified build of src/input.css -> dist/output.css
npm run watch   # rebuild on change while editing
python3 -m http.server 8000   # preview at http://localhost:8000
```

GitHub Pages runs no build step, so commit `dist/output.css` along with any change to `index.html` or `src/input.css`. Tailwind only emits classes it finds in `index.html`, so if you add a class without rebuilding, it won't render on the live site.

Tailwind v4 has no `tailwind.config.js`; all configuration lives in `src/input.css`:
- `@import "tailwindcss" source(none)` plus `@source "../index.html"` limits class scanning to the page itself (otherwise it would also pick up class-like text in files like this one).
- `@theme` pins the v3 hex values of the palette colors the page uses. v4's defaults are oklch and render slightly differently. Gradients use the `/srgb` modifier (`bg-linear-to-r/srgb`) for the same reason. Keep both if you touch the gradient, or the colors will shift.
- `package.json` has an npm `overrides` entry that forces `@parcel/watcher` to 2.5.6. The Tailwind CLI pins 2.5.1, which pulls in `micromatch` → `braces`, and every version of `braces` has an open security advisory. 2.5.6 uses `picomatch` instead. You can drop the override once a Tailwind CLI release depends on a watcher ≥ 2.5.6 itself.

## Design

The page uses a blue-800 gradient with white text. Desktop (`md` and up) runs left-to-right to sky-300; phones run diagonally to sky-400 (and get a blue-tinted corner glow) because the near-white sky end looked washed out on a small screen. The content sits directly on the gradient (an earlier floating glass panel was rejected as too heavy):
- `.pill` / `.pill-primary` are the glass-style link buttons.
- `.caustics` holds blurred radial-gradient shapes that drift slowly behind the content. They animate `transform` only, to keep them cheap.
- The name uses the Unbounded font from Google Fonts, requested with `text=Vibhu%20Agrawal` so only those glyphs download. If the headline text changes, update that `text=` parameter to match.
- All motion is disabled under `prefers-reduced-motion`. Keep the page lightweight: it ships no JavaScript, and should not pull in frameworks or icon libraries.

## Assets

- `index.html` links the resume at `src/Resume.pdf`. That is the file to replace when the resume changes; most commits in this repo do only that.
- `Resume.pdf` at the repo root is an older copy that nothing links to. `src/Karthik Resume.pdf` is someone else's resume and is also not linked from the page.
- The headshot is `src/images/headshot.jpg`.
