# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal website, live at https://vibhuagrawal.dev (custom domain set by `CNAME`; GitHub Pages serves the `main` branch, and vibhuag.github.io redirects there). There is no framework or bundler: the site is two static pages, `index.html` (home) and `projects/index.html` (served at `/projects/`), styled with Tailwind CSS v4 utilities plus custom component classes in `src/input.css`. There are no tests or linters.

## Build

```sh
npm install
npm run build   # minified build of src/input.css -> dist/output.css
npm run watch   # rebuild on change while editing
python3 -m http.server 8000   # preview at http://localhost:8000
```

GitHub Pages runs no build step, so commit `dist/output.css` along with any change to `index.html` or `src/input.css`. Tailwind only emits classes it finds in the HTML pages, so if you add a class without rebuilding, it won't render on the live site.

Tailwind v4 has no `tailwind.config.js`; all configuration lives in `src/input.css`:
- `@import "tailwindcss" source(none)` plus one `@source` line per page limits class scanning to the HTML pages (otherwise it would also pick up class-like text in files like this one). When you add a page, add an `@source` line for it.
- `@theme` pins the v3 hex values of the palette colors the page uses. v4's defaults are oklch and render slightly differently. Gradients use the `/srgb` modifier (`bg-linear-to-r/srgb`) for the same reason. Keep both if you touch the gradient, or the colors will shift.
- `package.json` has an npm `overrides` entry that forces `@parcel/watcher` to 2.5.6. The Tailwind CLI pins 2.5.1, which pulls in `micromatch` → `braces`, and every version of `braces` has an open security advisory. 2.5.6 uses `picomatch` instead. You can drop the override once a Tailwind CLI release depends on a watcher ≥ 2.5.6 itself.

## Design

The page uses a blue-800 gradient with white text. Desktop (`md` and up) runs left-to-right to sky-300; phones run diagonally to sky-400 (and get a blue-tinted corner glow) because the near-white sky end looked washed out on a small screen. The content sits directly on the gradient (an earlier floating glass panel was rejected as too heavy):
- `.pill` / `.pill-primary` are the glass-style link buttons on the home page.
- `.site-nav` is the menu bar (Home / Projects). There's no templating, so its markup is copied into each page; keep the copies in sync. `aria-current="page"` marks the current page, and the CSS styles that link as active.
- `.glass-card` is the liquid-glass project card on the projects page: `backdrop-filter` blur, plus a bright rim drawn with a mask in `::before`. To add a project, copy the Tal `<a class="glass-card">` block (preview image, then `.card-body`); the grid fills from the top-left, as many cards per row as fit at 14rem minimum (four on desktop, one on phones).
- Pages use relative paths (`dist/output.css` from the root, `../dist/output.css` from `projects/`) so they work both on the live domain and on a local server.
- `.caustics` holds blurred radial-gradient shapes that drift slowly behind the content. They animate `transform` only, to keep them cheap.
- The home-page name and project card titles are set in bold Helvetica (as on NYC subway signage) via the stack `"Helvetica Neue", Helvetica, Arial, sans-serif`: real Helvetica on Apple devices, Arial elsewhere. The site loads no web fonts.
- All motion is disabled under `prefers-reduced-motion`. Keep the page lightweight: it ships no JavaScript, and should not pull in frameworks or icon libraries.

## Assets

- `index.html` links the resume at `src/Resume.pdf`. That is the file to replace when the resume changes; most commits in this repo do only that.
- `Resume.pdf` at the repo root is an older copy that nothing links to. `src/Karthik Resume.pdf` is someone else's resume and is also not linked from the page.
- The headshot is `src/images/headshot.jpg`. Project card images go in `src/images/projects/`, named after the project, as WebP (convert with `cwebp -q 82`). Cards show the whole image uncropped, so its shape sets the card's height; set the `<img>` `width`/`height` attributes to the file's real pixel size so the layout doesn't jump while it loads.
