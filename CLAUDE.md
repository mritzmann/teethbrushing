# CLAUDE.md

Guidance for working on this repo in future sessions.

## What this is

A German-language teeth-brushing timer PWA for iOS. Guides the user through
6 brushing zones (front/back/chewing surfaces × upper/lower jaw) in random
order, each for a configurable duration (default 30s). Single-page app,
no backend, no build step.

## Architecture

- **`index.html`** — everything: markup, CSS, JS, in one file, intentionally
  (the original request was "one file, no build step"). Keep it that way
  unless the user asks for a different structure.
- **`manifest.json`**, **`sw.js`**, **`icon-*.png`** — added later for iOS
  "Add to Home Screen" support. `sw.js` does simple cache-first offline
  caching of the app shell (see `ASSETS` array — update it if you add files).
- No package.json, no npm, no framework. Don't introduce one without asking.

## Key implementation details

- **Zones**: defined in the `ZONES` array in the `<script>` block. Each has
  `jaw` (`upper`/`lower`) and `part` (`outer`/`inner`/`chew`), which map to
  German labels the user specified verbatim (`Vorderseite unten` etc.).
  Order is shuffled (Fisher–Yates) on every `startSession()`.
- **Mouth diagram**: a schematic SVG, not real anatomy. Two half-circles
  (upper = top semicircle, lower = bottom semicircle) built from 3 concentric
  radius bands each (`inner`/`chew`/`outer`), generated at runtime via
  `bandPathD()` using polar coordinates (`RADII`, `CX`, `CY_UPPER`,
  `CY_LOWER` constants). The active zone's band gets the `.active` class;
  a small pulsing dot (`#brushMarker`, styled like a Maps "current
  location" marker) sweeps linearly across the active band over the zone's
  duration — this was a deliberate revision (see git log) from an earlier
  back-and-forth sweep, which the user found confusing.
- **Timer**: `requestAnimationFrame` loop with delta-time accumulation
  (`state.elapsed`), not `setInterval` — avoids drift. Pausing just stops
  advancing `state.elapsed`; no separate pause timer needed.
- **Pause UI**: do NOT reintroduce a full-screen "Paused" overlay covering
  the controls — an earlier version did this and it blocked the resume
  button (z-index/click-blocking bug, fixed in commit `1ea5664`). The
  current approach is a small non-interactive status pill
  (`pointer-events:none`) plus an `.is-paused` class that freezes the CSS
  pulse/marker animations via `animation-play-state`.
- **iOS PWA feel**: `navigator.vibrate()` on zone change, a short WebAudio
  beep (no audio files), and the Screen Wake Lock API while a session is
  running (all feature-detected, fail silently if unsupported).

## Design system (as of the iOS redesign, commit `915c4bb`)

The UI intentionally mimics native iOS, **dark mode only** (no
`prefers-color-scheme` light variant — see rationale below):

- System font stack (`-apple-system, ...`), no custom fonts.
- iOS dark system-color palette as CSS variables in `:root` (`--bg`,
  `--card`, `--label`, `--label2`/`--label3` for secondary/tertiary text,
  `--separator`, `--fill`, single `--accent` green tint used consistently
  for all interactive/active elements — like a single app-tint color in
  native iOS apps).
- Large-title nav row on the idle screen only; active/done screens have no
  chrome (mirrors Apple's Clock app Timer tab, which also has minimal chrome
  while running).
- Grouped-list "card" rows for settings/info (`.card` + `.list-row`).
- Settings are a bottom sheet (`#settingsPanel`) with a grabber handle and
  a trailing "Fertig" button, not a centered modal.
- Session controls are two circular buttons (neutral "Abbrechen" / accent
  "Pause"–"Weiter"), directly modeled on the Clock app's Timer running
  screen.
- **Why dark-only**: the app is realistically used in a bathroom, often at
  night. Supporting light mode too would require picking a second
  `apple-mobile-web-app-status-bar-style`/`theme-color` pair, and
  `black-translucent` (needed for the immersive dark look) gives
  low-contrast white status bar icons on a light background. Dark-only
  avoids that whole class of bug. If the user asks for light mode, budget
  time to also solve the status bar contrast problem, not just add a
  media query.

## Icons

`icon-32/180/192/512.png` were generated with a throwaway Python script
(stdlib only: `struct` + `zlib`, no Pillow/ImageMagick/browser rendering —
the user asked not to install tools). The script isn't committed; the
design is a simple flat background + white rounded-rect "tooth" silhouette
(crown + two roots) + a small accent-colored circle. If icons need to
change, regenerate similarly (pure-Python PNG encoder) rather than adding
an image-processing dependency, unless the user says otherwise.

## Deployment

- Remote: `git@github.com:mritzmann/teethbrushing.git`, branch `main`.
- Hosted via GitHub Pages, branch `main` / root — this was enabled once
  manually in the GitHub web UI (Settings → Pages), since `gh` CLI is not
  installed and the user asked to use plain `git` only. There's no CI/build
  step; pushing to `main` is enough, Pages serves the files as-is.
- No `gh` CLI in this environment — don't assume it's available for PRs,
  issues, or Pages API calls.

## Working conventions observed in this repo

- The user writes to Claude in German; code, comments, commit messages,
  and docs are in English (labels/copy in the app itself are German, as
  the app's target audience is German-speaking).
- Keep commits scoped to one change; the user reviews and explicitly asks
  for commit+push each time rather than batching.
- Don't install tools/browsers/CLIs without asking — this environment has
  Python 3 and Chrome available already; prefer using what's there.
