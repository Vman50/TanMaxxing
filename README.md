# TanMaxxing ☀️

**Optimize Your Glow** — a mobile-first web app that tracks the UV index for your
location and helps you time tanning sessions to build a tan while staying clear of
sunburn.

It's a self-contained `index.html` (plus a tiny `sw.js` service worker for offline use) —
no build step and no dependencies.

## Use it

**▶️ Live app: https://vman50.github.io/TanMaxxing/**

Just open that link on your phone or desktop — nothing to install. On mobile you can
"Add to Home Screen" for an app-like, offline-capable experience.

When you first open it, set your location either by **allowing location access** or by
**typing a city / place** (or `lat, lng`) in onboarding or **Settings → Location**.

## Features

- **Live UV index** for your location — set by GPS or by typing a city/place — with an
  hourly UV curve, a 7-day outlook, and the best tanning window of the day.
- **Session timer** — a dial that tracks active sun time and warns you before you hit
  your burn limit.
- **Tan progress** — a visual "tan gem" and goal bar showing how your tan is building,
  holding, or fading over time.
- **Personalized to your skin** — a short onboarding quiz estimates your Fitzpatrick
  skin type and adjusts burn times and recommendations.
- **SPF & products** — log sunscreen/products and the app factors effective SPF into
  safe-exposure estimates.
- **Session log** — history of past sessions on the Log screen.
- Installable as a PWA on mobile, with offline-friendly local storage.

## Run it locally (for development)

You only need this if you want to hack on the app — end users should just use the
[live link](https://vman50.github.io/TanMaxxing/) above.

1. Clone the repo:
   ```bash
   git clone https://github.com/Vman50/TanMaxxing.git
   cd TanMaxxing
   ```
2. Serve it over HTTP so the service worker, geolocation, and "install to home screen"
   work (these are disabled on `file://`):
   ```bash
   python3 -m http.server 8000
   # then visit http://localhost:8000
   ```
   Opening `index.html` directly still works for most things, just without the PWA bits.

### OpenUV API key (optional)

For the most accurate, real-time UV index the app uses [OpenUV](https://www.openuv.io/).
Grab a free API key (50 lookups/day), then paste it into **Settings → OpenUV API key**.
The key is stored only on your device in `localStorage` and is never sent anywhere else.

Without a key, the app falls back to the keyless [Open-Meteo](https://open-meteo.com/)
UV forecast.

## Data & privacy

All your data — sessions, skin profile, products, and any API key — stays in your
browser's `localStorage`. Nothing is uploaded to a server owned by this project. The app
calls third-party APIs (OpenUV, Open-Meteo, and BigDataCloud for the location name) only
to fetch UV and location data.

## Tech

Plain HTML, CSS, and vanilla JavaScript in one file. No frameworks, no bundler.

## Disclaimer

TanMaxxing provides estimates for informational purposes only and is **not** medical
advice. UV exposure carries real risks including sunburn and skin cancer. Always use
appropriate sun protection and consult a professional for guidance on safe sun exposure.
