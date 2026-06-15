# TanMaxxing ☀️

**Optimize Your Glow** — a mobile-first web app that tracks the UV index for your
location and helps you time tanning sessions to build a tan while staying clear of
sunburn.

It's a single, self-contained `index.html` — no build step, no dependencies, no server
required. Just open it in a browser.

## Features

- **Live UV index** for your current location with an hourly UV curve and the best
  tanning window of the day.
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

## Getting started

1. Clone the repo:
   ```bash
   git clone https://github.com/Vman50/TanMaxxing.git
   cd TanMaxxing
   ```
2. Open `index.html` in a browser. For full PWA behavior (geolocation, install to home
   screen), serve it over HTTP, e.g.:
   ```bash
   python3 -m http.server 8000
   # then visit http://localhost:8000
   ```
3. Allow location access when prompted.

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
