# Changelog

## [2026-06-15] — Per-Side Timer, Stopwatch Mode & Copy Polish

### Added
- **Per-side timer memory** — the dial now tracks elapsed time per body side independently. Each side's accumulated time is preserved across pauses and flip operations, so you always know exactly how long each side has been exposed.
- **Stopwatch mode** — the timer now runs as a stopwatch; the calculated safe time is shown as a suggestion rather than a hard limit. The ring fills to the suggestion, turns red when it's reached, and keeps counting so you stay in control.
- **Pre-fill panel** — a compact controls panel appears above the pause row showing each side's target, with +/− adjustments, letting you dial in per-side goals before or during a session.
- **Reset timer button** — a "↺ Reset timer" button appears while a session is running or paused, so you can scrap and restart without leaving the screen.
- **Per-side time fields in Add/Edit sheet** — the manual log sheet now shows individual time-per-side inputs (one field per active side) instead of a single total-minutes field, with the total derived automatically.
- **Mode selector as a slider** — the Build / Protect mode toggle is now a smooth range slider with labelled endpoints and an animated fill, replacing the tab-button segmented control.
- **"Base tan" shade label** — the first non-zero tan level now reads "Base tan" instead of "Faint glow" to better reflect what's actually building.
- **GitHub Actions deploy workflow** — pushes to `main` auto-deploy to the GitHub Pages root (production); pushes to `dev` auto-deploy to the `/dev` subdirectory (preview).

### Changed
- **Copy: "glow" → "tan" throughout** — every user-facing string, label, card heading, and aria label that said "glow" has been updated to "tan" for a more direct, masculine tone. Affected strings include the page title ("Build Your Tan"), share button label, chart heading, weekly summary ("Peak tan"), tan-forecast subhead, Settings goal label, science explanation, onboarding headline, and the tan-note placeholder.
- **Onboarding CTA** — final onboarding button changed from "Start glowing" to "Start tracking".
- **Sun-times label** — "golden" window label changed to "peak" in the stats bar.
- **Dial sub-label** — while the timer is running the secondary label now shows the suggested side time ("~Nm suggested") rather than a countdown.
- **Dial tap area** — when multiple sides are active, shows the running total across all sides ("total: MM:SS") instead of a burn-percentage estimate.

### Fixed / Internal
- **Storage namespace renamed** — all `localStorage` keys migrated from the old `bask.*` prefix to `tanmax.*`. A one-time migration runs on startup to copy any existing `bask.*` data into `tanmax.*` and clean up the old keys, so no user data is lost on upgrade.
- **Timer state extended** — `sideTimes` map added to the persisted timer object so per-side totals survive page reloads mid-session.

---

## [2026-06-15] — Science-Based SPF, UI Polish & PWA Icons

### Added
- Science-based SPF model — effective SPF calculated as `√(label)` to match real-world coverage (Faurschou & Wulf); high-SPF labels are discounted, low-SPF barely changes.
- Auto-labelled product chips (Tanning Oil / Sunscreen) derived from product kind.
- Generated PWA icons from canvas at runtime (192/512 any + 512 maskable); favicon and apple-touch-icon all share the same brand gem.
- Manual location entry — type a city or `lat, lng` pair in Settings or onboarding; geocoded via Open-Meteo Geocoding API.
- Pull-to-refresh on the Today screen.
- Swipe-down-to-dismiss on all bottom sheets, integrated with browser history (back button closes sheets).
- Haptic feedback on key taps via `navigator.vibrate`.
- AM/PM time display throughout.
- Vitamin D stat on the stats bar.

### Changed
- Build/Max mode parity fixed — safe-time calculation consistent between modes.
- PWA UX polish: press-scale feedback, cross-fade screen transitions, reduced-motion support.

---

## [2026-06-14] — Initial Release

- UV index tracking via OpenUV (key required) with Open-Meteo as a keyless fallback.
- Skin-type onboarding (Types I–VI) with MED-based burn-time calculation.
- Session timer with body-side rotation (Front/Back/Left/Right).
- Daily UV budget ring and tan-progress tracking with ~18-day fade model.
- 7-day UV forecast card and tan-fade forecast.
- SPF product manager with reapplication countdown.
- Reflective environment multipliers (beach sand, snow, water).
- Sunburn lockout (recovery mode) with 24-hour cooldown.
- Weekly summary stats and session log with edit support.
- Tan share card (Web Share API + canvas).
- Dark / light / auto theme with system-preference detection.
- Service worker for offline caching and Add-to-Home-Screen.
