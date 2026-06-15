# CLAUDE.md

Guidance for working in this repository.

## Personal

- Always greet the user by name — **Vince** — at the start of every chat.

## What this is

TanMaxxing ("Optimize Your Glow") is a single-file, mobile-first **PWA** that helps
users track UV exposure and time their tanning sessions to build a tan while avoiding
sunburn. The entire app — HTML, CSS, and JavaScript — lives in **`index.html`**.

## Architecture

- **One file, no build step.** Everything is inline in `index.html`: a `<style>` block
  for theming/layout and a single `<script>` block (vanilla JS, no frameworks, no
  bundler). Open the file in a browser to run it; deploy by serving the static file.
- **Mobile-first / PWA.** Designed for a phone-width frame (`.app` max-width 460px),
  uses `env(safe-area-inset-*)`, Apple web-app meta tags, and a data-URI touch icon.
- **State** is a single in-memory `state` object persisted to `localStorage` under keys
  prefixed `tanmaxxing-data-` (see the `ls()` / `store()` helpers).
- **Theming** via `html[data-theme="light|dark"]` and CSS custom properties (`--bg`,
  `--accent`, etc.). Keep new colors as tokens, not hard-coded values.
- **Two screens** — `#screen-today` and `#screen-log` — toggled by adding/removing the
  `.hidden` class. Modals are bottom sheets (`#addSheet`, `#settingsSheet`,
  `#productsSheet`, `#sciSheet`, `#confirmFinishSheet`) opened via `openSheet()`.

## External services

- **OpenUV** (`api.openuv.io`) — primary UV index + sun times. Requires a free user API
  key (`x-access-token`, 50 lookups/day) entered in Settings; stored only in
  `localStorage` on the device. See `fetchOpenUV()`.
- **Open-Meteo** (`api.open-meteo.com`) — keyless UV forecast fallback. See
  `fetchOpenMeteo()`.
- **BigDataCloud** (`api.bigdatacloud.net`) — keyless reverse geocoding for the location
  label. See `reverseGeocode()`.
- **Geolocation** via the browser `navigator.geolocation` API (`getLocation()`).
- **Google Fonts** (Fraunces, Inter) loaded from CDN.

No API keys are committed to the repo. Never hard-code a key in `index.html`.

## Conventions

- Match the existing terse, single-line vanilla-JS style and the CSS-token approach.
- UI is built imperatively by `render*` / `build*` functions; after changing state call
  the relevant `render*` (e.g. `renderAll()`, `renderToday()`, `renderTan()`).
- Keep it dependency-free and single-file unless there's a strong reason to split.

## Testing

There is no test suite or tooling. Verify changes by opening `index.html` in a browser
(or a simple static server) and exercising the Today and Log screens, the session timer,
onboarding quiz, and Settings.
