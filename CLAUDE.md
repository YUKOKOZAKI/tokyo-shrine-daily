# CLAUDE.md

## Project Overview

**Tokyo Shrine Daily** (東京神社めぐり) is a static Progressive Web App that displays one Tokyo shrine per day. Users can browse a curated collection of 24 shrines, learning about their history, features, worship benefits, and practical visit information.

- **Language**: Content is in Japanese; code identifiers are in English
- **Stack**: Vanilla HTML/CSS/JavaScript — no frameworks, no build tools, no dependencies
- **Deployment**: GitHub Pages via `.github/workflows/pages.yml` on push to `main`

## File Structure

```
.
├── index.html        # Entire application (HTML + inline CSS + inline JS, ~990 lines)
├── manifest.json     # PWA manifest (app name, theme, icons)
├── sw.js             # Service Worker (cache-first with network update strategy)
├── icon.svg          # SVG app icon (torii gate)
├── CLAUDE.md         # This file
└── .github/
    └── workflows/
        └── pages.yml # GitHub Pages deployment workflow
```

This is a **single-file application**. All markup, styles, and logic live in `index.html`.

## Architecture of index.html

| Lines     | Section                            |
|-----------|------------------------------------|
| 1–13      | HTML head, meta tags, PWA config   |
| 14–349    | Inline CSS (all styles)            |
| 350–397   | HTML body structure                |
| 399–862   | `shrines` data array (24 objects)  |
| 864–872   | `getTodayIndex()` — picks shrine based on day-of-year |
| 874–882   | `formatDate()` — Japanese date formatting |
| 884       | `currentIndex` state variable      |
| 886–960   | `renderShrine(index)` — builds shrine card DOM via innerHTML |
| 962–967   | `navigate(direction)` — prev/next navigation |
| 969–978   | `openLightbox(src)` — image lightbox |
| 980–987   | Initialization and Service Worker registration |

### Shrine Data Schema

Each shrine object in the `shrines` array has:

```javascript
{
    name: "明治神宮",              // Shrine name (kanji)
    reading: "めいじじんぐう",      // Phonetic reading (hiragana)
    location: "東京都渋谷区...",    // Full address
    tags: ["初詣日本一", ...],     // Characteristic tags
    mainImage: "https://...",      // Unsplash hero image URL
    gallery: ["https://...", ...], // Additional Unsplash image URLs
    history: "...",                // History/origin text
    features: "...",               // Highlights/features text
    worship: "...",                // Benefits/worship text
    info: {                        // Key-value info table
        "創建": "1920年",
        "御祭神": "明治天皇",
        // ...
    },
    mapQuery: "明治神宮"           // Google Maps search query
}
```

## Development

### Running Locally

No build step is required. Serve the root directory with any static file server:

```sh
# Python
python3 -m http.server 8000

# Node (npx)
npx serve .
```

Then open `http://localhost:8000` in a browser.

### No Build Tools / No Dependencies

- No `package.json`, no `node_modules`, no bundler
- No TypeScript, no CSS preprocessor, no linter configured
- No test framework or test files

### Deployment

Pushing to `main` triggers the GitHub Actions workflow in `.github/workflows/pages.yml`, which deploys the entire root directory to GitHub Pages.

## Service Worker (`sw.js`)

- Cache name: `shrine-v1`
- Strategy: Stale-while-revalidate — serves cached content first, fetches network update in background
- Cached assets: `./`, `./index.html`, `./manifest.json`, `./icon-192.png`, `./icon-512.png`
- Old caches are cleaned up on activation

**Note**: `icon-192.png` and `icon-512.png` are listed in the SW cache but do not exist as files. The actual icon is `icon.svg`. This is a known inconsistency.

## Code Conventions

### Naming

- **Functions**: camelCase (`getTodayIndex`, `renderShrine`, `openLightbox`)
- **Variables**: camelCase (`currentIndex`, `dayOfYear`)
- **CSS classes**: kebab-case (`.shrine-card`, `.nav-btn`, `.gallery`)
- **CSS IDs**: camelCase (`#shrineCard`, `#dateBadge`, `#lightbox`)

### Style Patterns

- All code is inline within `index.html` (no external JS/CSS files)
- DOM rendering uses template literals assigned to `innerHTML`
- Event handlers use inline `onclick` attributes
- No module system, no imports/exports
- Minimal comments — code is self-documenting through descriptive function names

### Design System

- **Primary color**: `#8b2635` (deep shrine red)
- **Background**: `#f5f0e8` (warm cream)
- **Title font**: Noto Serif JP (Google Fonts)
- **Body font**: Noto Sans JP (Google Fonts)
- **Responsive breakpoint**: 600px max-width
- **Images**: All sourced from Unsplash with URL-based sizing parameters

## Guidelines for AI Assistants

1. **Single-file architecture**: All changes to the app happen in `index.html`. Do not split into separate files unless explicitly asked.
2. **No build tools**: Do not introduce `package.json`, bundlers, or build steps unless explicitly requested.
3. **Japanese content**: Shrine data is in Japanese. Preserve Japanese text accuracy when editing content.
4. **Image URLs**: All images are external Unsplash URLs. Do not reference local image files.
5. **PWA consistency**: If changing cached assets, update both `sw.js` and `manifest.json`.
6. **Service Worker versioning**: Bump the cache name (e.g., `shrine-v1` → `shrine-v2`) when changing cached resources so clients pick up updates.
7. **Keep it simple**: This project values simplicity. Avoid introducing frameworks, libraries, or unnecessary abstractions.
8. **Accessibility**: Maintain semantic HTML structure (header, main, footer, nav) and image alt text.
