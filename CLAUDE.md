# CLAUDE.md — Tokyo Shrine Daily (東京神社めぐり)

## Project overview

A Progressive Web App (PWA) that surfaces one Tokyo shrine per day. Built as a
single, self-contained HTML file with no build step, no dependencies, and no
server. The app is deployed to GitHub Pages and installable on mobile devices.

---

## Repository layout

```
tokyo-shrine-daily/
├── index.html        # Entire application (HTML + CSS + JS in one file)
├── manifest.json     # PWA manifest (name, icons, display mode, theme colour)
├── sw.js             # Service worker (cache-first strategy for offline use)
├── icon.svg          # App icon — torii gate SVG (also used as favicon)
└── .github/
    └── workflows/
        └── pages.yml # GitHub Actions: deploys on push to `main`
```

There is no `package.json`, no bundler, no transpiler, no test framework.
Everything runs directly in the browser.

---

## Architecture

### Single-file app (`index.html`)

The app is entirely contained in `index.html`:

- **CSS** (lines 14–369): inlined `<style>` block; no external stylesheet except
  Google Fonts (`Noto Serif JP` and `Noto Sans JP`).
- **HTML** (lines 371–396): minimal skeleton — a `<header>`, a `<div
  class="container">` holding the shrine card, a `<footer>`, and a lightbox
  overlay.
- **JavaScript** (lines 398–988): inlined `<script>` block containing:
  - the `shrines` data array (20 shrine objects)
  - `getTodayIndex()` — day-of-year rotation algorithm
  - `formatDate()` — Japanese date formatter
  - `renderShrine(index)` — builds shrine card HTML via string interpolation
  - `navigate(direction)` — prev/next buttons
  - `openLightbox(src)` / click-to-close — photo lightbox
  - Service Worker registration

### Shrine data structure

Each entry in the `shrines` array is a plain object:

```js
{
  name: "明治神宮",              // Kanji shrine name
  reading: "めいじじんぐう",     // Hiragana reading
  location: "東京都...",         // Full Japanese address
  tags: ["...", "..."],          // Short descriptive tags (shown as pills)
  mainImage: "https://...",      // Unsplash hero image URL (800×500)
  gallery: ["https://...", ...], // Unsplash gallery images (400×300, 2 items)
  history: "...",                // Historical background paragraph (Japanese)
  features: "...",               // Notable features paragraph (Japanese)
  worship: "...",                // Deities / blessings paragraph (Japanese)
  info: {                        // Key-value info table rows
    "創建": "...",
    "御祭神": "...",
    "例大祭": "...",
    "参拝時間": "...",
    "アクセス": "..."
  },
  mapQuery: "..."                // String passed to Google Maps search URL
}
```

### Daily rotation algorithm

```js
function getTodayIndex() {
    const now = new Date();
    const start = new Date(now.getFullYear(), 0, 0);
    const diff = now - start;
    const oneDay = 1000 * 60 * 60 * 24;
    const dayOfYear = Math.floor(diff / oneDay);
    return dayOfYear % shrines.length;
}
```

`dayOfYear` is computed from January 0 (i.e. Dec 31 of prior year) so day 1 of
the year maps to index 1, not 0. The result wraps modulo the number of shrines
(currently 20), so the cycle repeats roughly every 20 days.

### Service worker (`sw.js`)

- Cache name: `shrine-v1`
- Strategy: **stale-while-revalidate** — returns cached response immediately
  while fetching a fresh copy in the background.
- Cached assets: `./`, `./index.html`, `./manifest.json`, `./icon-192.png`,
  `./icon-512.png`.
- On `activate`, old caches whose key ≠ `shrine-v1` are deleted.

> **Note:** `icon-192.png` and `icon-512.png` are listed in the service worker
> but do not exist in the repo (the only icon is `icon.svg`). Cache installation
> silently fails for missing assets, so this does not break the app but may
> cause console errors.

---

## Design conventions

| Token | Value | Usage |
|-------|-------|-------|
| Brand red | `#8b2635` / `#c41e3a` | Header gradient, shrine name, borders, tags |
| Background | `#f5f0e8` | Page background (warm off-white) |
| Card | `#ffffff` | Shrine card background |
| Divider | `#f0ebe3` | Section borders |
| Body text | `#444` | Content paragraphs |
| Muted text | `#999` | Reading, counter, footer |

**Typography**
- `Noto Serif JP` — headings, shrine names, section titles (loaded via Google Fonts)
- `Noto Sans JP` — body, buttons, UI elements

**Breakpoint:** `@media (max-width: 600px)` for mobile adjustments (single-column
gallery, smaller hero image, reduced font sizes, tighter padding).

---

## How to add a new shrine

1. Open `index.html` and find the `const shrines = [` array (line 399).
2. Append a new object following the shrine data structure above.
3. Use an Unsplash URL with `?w=800&h=500&fit=crop` for `mainImage` and
   `?w=400&h=300&fit=crop` for gallery images.
4. All text content (`history`, `features`, `worship`, info values) should be
   written in Japanese.
5. Set `mapQuery` to the most recognisable search string for Google Maps.
6. No build step required — save and open in a browser.

---

## Development workflow

### Local development

There is no dev server required. Open `index.html` directly in a browser:

```bash
open index.html          # macOS
xdg-open index.html      # Linux
```

For service worker and PWA features to work, serve over HTTP:

```bash
npx serve .              # or python3 -m http.server 8080
```

### Making changes

- All content, styles, and logic live in `index.html`.
- `manifest.json` — update only for PWA metadata changes (name, colours, icons).
- `sw.js` — update `CACHE_NAME` (e.g. `shrine-v2`) whenever cached assets
  change, to force a cache refresh in existing installs.
- `icon.svg` — SVG torii gate; edit directly if a new icon is needed.

### No tests

There is no test suite. Verify changes by loading the page in a browser.

---

## Deployment

The app deploys automatically via GitHub Actions on every push to `main`.

**Workflow:** `.github/workflows/pages.yml`

Steps:
1. `actions/checkout@v4` — check out the repository
2. `actions/configure-pages@v5` — configure GitHub Pages
3. `actions/upload-pages-artifact@v3` — upload the entire repo root as the
   Pages artifact
4. `actions/deploy-pages@v4` — deploy to GitHub Pages

Required repository permissions: `contents: read`, `pages: write`,
`id-token: write`.

The deployed URL follows the pattern:
`https://<org>.github.io/<repo>/`

---

## Known issues / caveats

- The service worker lists `icon-192.png` and `icon-512.png` as cached assets
  but only `icon.svg` exists. Update `sw.js` if PNG icons are added, or remove
  the PNG entries if they are not needed.
- The `pages.yml` workflow deploys on push to `main`, but the default branch in
  this repo appears to be `master`. Adjust the `branches` filter in the workflow
  if the default branch name changes.
- All shrine images are hosted on Unsplash and loaded at runtime; they will not
  be available offline via the service worker cache (only HTML/manifest are
  cached locally).
- The daily index algorithm uses `new Date()` in the user's local timezone, so
  the "shrine of the day" will differ for users in different timezones.
