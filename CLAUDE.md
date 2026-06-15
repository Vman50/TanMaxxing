# CLAUDE.md

Guidance for working in this repository.

## Personal

- Always greet the user by name — **Vince** — at the start of every chat.

## What this is

TanMaxxing ("Optimize Your Glow") is a mobile-first **PWA** that helps
users track UV exposure and time their tanning sessions to build a tan while avoiding
sunburn. The app — HTML, CSS, and JavaScript — lives in **`index.html`**, with a small
companion **`sw.js`** service worker for offline caching.

## Architecture

- **One main file, no build step.** Everything is inline in `index.html`: a `<style>` block
  for theming/layout and a single `<script>` block (vanilla JS, no frameworks, no
  bundler). Open the file in a browser to run it; deploy by serving the static files.
- **Service worker (`sw.js`).** The one exception to single-file: a minimal worker that
  caches the app shell (`./`, `./index.html`) for offline use and proper Add-to-Home-Screen,
  network-first for navigations. Registered from `index.html` (`navigator.serviceWorker.register("sw.js")`);
  registration is guarded in try/catch so `file://` still works. It must be served next to
  `index.html`. It also handles `notificationclick` to focus the app.
- **Mobile-first / PWA.** Designed for a phone-width frame (`.app` max-width 460px),
  uses `env(safe-area-inset-*)` and Apple web-app meta tags. App icons are generated at
  runtime from a `<canvas>` (the brand "gem") by `drawAppIcon(size,maskable)` — favicon,
  `apple-touch-icon`, and the manifest's 192/512 `any` + 512 `maskable` PNG icons all come
  from it. A static inline SVG `apple-touch-icon` in `<head>` is the pre-JS fallback.
- **State** is a single in-memory `state` object persisted to `localStorage` under keys
  prefixed `bask.` (e.g. `bask.sessions`, `bask.skin`, `bask.loc`) via the `Store.read()` /
  `Store.write()` helpers, which fall back to an in-memory `mem` object if `localStorage`
  is unavailable. The OpenUV key is base64-encoded (`btoa`/`atob`) under `bask.openuvKey`.
  Other keys: `bask.sides` (body sides the timer rotates through — `Front/Back/Left/Right`),
  `bask.env` (reflective-environment multipliers, see `ENV`/`envMult()`), `bask.recoveryUntil`
  (sunburn-lockout timestamp), `bask.productAppliedAt` (sunscreen re-application timer), and
  `bask.weekOpen` (weekly-summary expanded state). Sessions may carry `sides` (array) and
  `burned` (bool) fields.
- **Theming** via `html[data-theme="light|dark"]` and CSS custom properties (`--bg`,
  `--accent`, etc.). Keep new colors as tokens, not hard-coded values.
- **Two screens** — `#screen-today` and `#screen-log` — toggled by adding/removing the
  `.hidden` class. Modals are bottom sheets (`#addSheet`, `#settingsSheet`,
  `#productsSheet`, `#sciSheet`, `#confirmFinishSheet`) opened via `openSheet()`.
  Today also hosts: `#recoveryCard` (sunburn lockout), `#nudgeCard` (best-time-to-tan),
  `#budgetCard` (daily MED/UV budget), `#forecastCard` (7-day UV outlook), and the
  `#reapplyLine` countdown. Log hosts `#weekCard` (collapsible weekly summary) and
  `#tanfcCard` (tan-fade forecast). `#addSheet` doubles as the edit-session sheet (its
  title/fields are pre-filled via `openEdit()` when a log row is long-pressed; `editingId`
  tracks edit vs. add).

## External services

- **OpenUV** (`api.openuv.io`) — primary UV index + sun times. Requires a free user API
  key (`x-access-token`, 50 lookups/day) entered in Settings; stored only in
  `localStorage` on the device. See `fetchOpenUV()`.
- **Open-Meteo** (`api.open-meteo.com`) — keyless UV forecast fallback. See
  `fetchOpenMeteo()`.
- **BigDataCloud** (`api.bigdatacloud.net`) — keyless reverse geocoding for the location
  label. See `reverseGeocode()`.
- **Open-Meteo Geocoding** (`geocoding-api.open-meteo.com`) — keyless forward geocoding for
  manually-typed locations (a place name or a `lat, lng` pair). See `geocodePlace()` /
  `setManualLocation()`; wired to the Settings and onboarding "type a city" inputs as the
  second way to set location alongside `navigator.geolocation`.
- **Geolocation** via the browser `navigator.geolocation` API (`getLocation()`).
- **Google Fonts** (Fraunces, Inter) loaded from CDN.

Browser platform APIs (no network): **Web Notifications** for lock-screen timer alerts
(`requestNotify()` / `fireNotify()`, fired from the 2-min warning and `burnAlert()`),
**Web Share / canvas** for the "My glow today" share card (`shareGlow()` /
`buildShareCanvas()`, header share button), plus the existing Wake Lock + Web Worker timer.
Open-Meteo is fetched with `forecast_days=7`; `state.live.daily` holds per-day max/sun/hourly
for the 7-day forecast.

No API keys are committed to the repo. Never hard-code a key in `index.html`.

## Conventions

- Match the existing terse, single-line vanilla-JS style and the CSS-token approach.
- UI is built imperatively by `render*` / `build*` functions; after changing state call
  the relevant `render*` (e.g. `renderAll()`, `renderToday()`, `renderTan()`).
- Keep it dependency-free and single-file unless there's a strong reason to split.
- **Keep this file up to date.** Whenever a change alters something documented here —
  architecture, state keys, screens/sheets, external services, conventions, or testing —
  update `CLAUDE.md` in the same change so it never drifts from `index.html`.

## Testing

There is no test suite or tooling. Verify changes by opening `index.html` in a browser
(or a simple static server) and exercising the Today and Log screens, the session timer,
onboarding quiz, and Settings. The service worker, notifications, and Add-to-Home-Screen
only work over `http(s)` (a static server), not `file://`; the rest works either way.
A quick JS sanity check: extract the `<script>` body and pass it through `new Function(...)`
in Node to catch syntax errors before opening the page.
