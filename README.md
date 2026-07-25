# Zahnputz-Assistent

A guided teeth-brushing timer for iOS, installable as a home-screen web app.

Splits brushing into 6 zones (outer/inner/chewing surfaces, upper/lower jaw),
shown in random order, with a countdown ring and a mouth diagram that
highlights where to brush next. Zone duration defaults to 30s and is
configurable.

## Usage

Open `index.html` directly in a browser — no build step, no server, no
dependencies. On iOS, open it in Safari and use **Share → Add to Home
Screen** to install it as a standalone app (works offline via the
included service worker).

Live version (once GitHub Pages is enabled): `https://mritzmann.github.io/teethbrushing/`

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire app (markup, CSS, JS) |
| `manifest.json` | Web app manifest for "Add to Home Screen" |
| `sw.js` | Service worker, caches the app for offline use |
| `icon-*.png` | App icons, generated programmatically (see `CLAUDE.md`) |

See `CLAUDE.md` for implementation details and design decisions.
