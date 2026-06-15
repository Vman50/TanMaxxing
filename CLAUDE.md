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
  prefixed `tanmax.` (e.g. `tanmax.sessions`, `tanmax.skin`, `tanmax.loc`) via the `Store.read()` /
  `Store.write()` helpers, which fall back to an in-memory `mem` object if `localStorage`
  is unavailable. The OpenUV key is base64-encoded (`btoa`/`atob`) under `tanmax.openuvKey`.
  Other keys: `tanmax.sides` (body sides the timer rotates through — `Front/Back/Left/Right`),
  `tanmax.env` (reflective-environment multipliers, see `ENV`/`envMult()`), `tanmax.recoveryUntil`
  (sunburn-lockout timestamp), `tanmax.productAppliedAt` (sunscreen re-application timer), and
  `tanmax.weekOpen` (weekly-summary expanded state), and `tanmax.theme` (`auto`/`light`/`dark`
  appearance preference). Sessions may carry `sides` (array), `burned` (bool), and `spf`
  (effective-SPF multiplier captured at finalize, used to discount that session's contribution
  to the daily UV budget — see `todayDose()`) fields.
- **Theming** via `html[data-theme="light|dark"]` and CSS custom properties (`--bg`,
  `--accent`, etc.). Both light and dark token blocks are defined; `applyTheme()` resolves
  `state.theme` (Auto follows `prefers-color-scheme`) to the `data-theme` attribute and keeps
  the `#themeColorMeta` status-bar tint in sync. A tiny inline `<head>` script sets `data-theme`
  before first paint to avoid a flash. Appearance is switchable in Settings (`#themeSeg`).
  `themeColors()` (canvas wave/chart colors) branches on the active theme. Keep new colors as
  tokens, not hard-coded values. The iOS status bar uses `apple-mobile-web-app-status-bar-style`
  = `default` (readable on the light theme; tinted via `theme-color` on the dark theme).
- **Two screens** — `#screen-today` and `#screen-log` — toggled by adding/removing the
  `.hidden` class. Modals are bottom sheets (`#addSheet`, `#settingsSheet`,
  `#productsSheet`, `#sciSheet`, `#confirmFinishSheet`) opened via `openSheet()`.
  Sheets are **swipe-down-to-dismiss** (drag the `.grab` handle, or drag anywhere while the
  sheet is scrolled to top — see `makeSheetDraggable()`) and integrate with browser **history**:
  `openSheet()` pushes a history entry so the OS back button / back-swipe / `Escape` closes the
  sheet instead of leaving the app (`closeSheets()` pops it; a `popstate` handler keeps state in
  sync). A `haptic()` helper fires `navigator.vibrate` on key taps (no-op on iOS, which lacks the
  API). Interactive elements have press feedback (`:active` scale) and screens cross-fade on tab
  switch; all motion is disabled under `prefers-reduced-motion`.
  Today also hosts: `#recoveryCard` (sunburn lockout), `#nudgeCard` (best-time-to-tan),
  `#budgetCard` (daily MED/UV budget), `#forecastCard` (7-day UV outlook), and the
  `#reapplyLine` countdown. Log hosts `#weekCard` (collapsible weekly summary) and
  `#tanfcCard` (tan-fade forecast). `#addSheet` doubles as the edit-session sheet (its
  title/fields are pre-filled via `openEdit()` when a log row is tapped; `editingId`
  tracks edit vs. add).
- **Native-feel touches.** The Today screen has **pull-to-refresh** (drag down at scrollTop 0 to
  refetch UV; `#ptr` spinner indicator, wired near `makeSheetDraggable`). The status `↻` and
  `#ptr` spin while `setStatus("load",…)` is active. Glow percentages count up via
  `animateCount()`. Numeric inputs carry `inputmode`/`enterkeyhint`; `color-scheme` is set per
  theme so native form controls/scrollbars match. Body text is non-selectable (app-like) except
  inputs and the `.sci` science copy.

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

Browser platform APIs (no network): **Web Notifications** as *best-effort* foreground
reminders (`requestNotify()` / `fireNotify()`, fired from the 2-min warning and
`burnAlert()`) — NOT a reliable alarm. iOS suspends the Web Worker timer and cannot fire
scheduled background notifications, so the live timer is positioned as a glance reference
only; users are expected to set a native Clock-app timer for the real alert and log the
session afterward (the `+` quick-add pre-fills the planned minutes + current UV).
`burnAlert()` is therefore an in-app toast + soft notification with **no audible alarm**.
**Web Share / canvas** for the "My glow today" share card (`shareGlow()` /
`buildShareCanvas()`, header share button), plus the existing Wake Lock + Web Worker timer.
Open-Meteo is fetched with `forecast_days=7`; `state.live.daily` holds per-day max/sun/hourly
for the 7-day forecast.

No API keys are committed to the repo. Never hard-code a key in `index.html`.

## Development

**Run locally:** Serve the app over HTTP so PWA features (service worker, geolocation,
Add-to-Home-Screen) work:
```bash
python3 -m http.server 8000
# visit http://localhost:8000
```
Opening `index.html` directly still works for most features, just not PWA bits.

**Syntax check:** Extract the `<script>` block and validate it:
```bash
node -e "new Function($(sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d'))"
```

**Git workflow:** Always push to the `dev` branch first. Only push to `main` after a full
codebase review and explicit approval from Vince.

## Conventions

- Match the existing terse, single-line vanilla-JS style and the CSS-token approach.
- UI is built imperatively by `render*` / `build*` functions; after changing state call
  the relevant `render*` (e.g. `renderAll()`, `renderToday()`, `renderTan()`).
- Keep it dependency-free and single-file unless there's a strong reason to split.
- **Keep this file up to date.** Whenever a change alters something documented here —
  architecture, state keys, screens/sheets, external services, conventions, or development/testing —
  update `CLAUDE.md` in the same change so it never drifts from `index.html`.

## Testing

There is no test suite or tooling. Verify changes by opening `index.html` in a browser
(or a simple static server) and exercising the Today and Log screens, the session timer,
onboarding quiz, and Settings. The service worker, notifications, and Add-to-Home-Screen
only work over `http(s)` (a static server), not `file://`; the rest works either way.
A quick JS sanity check: extract the `<script>` body and pass it through `new Function(...)`
in Node to catch syntax errors before opening the page.
