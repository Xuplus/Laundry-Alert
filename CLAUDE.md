# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Single-file static web app (`index.html`) that tells users whether it's a good time to do laundry based on their local weather forecast. No build step, no dependencies, no package manager.

## Development

There is no build, lint, or test tooling. Edit `index.html` directly and open it in a browser to test. Push to `claude/laundry-alert-github-pages-UwUQR` (or `main`) to trigger a GitHub Pages deployment.

## Deployment

GitHub Pages is configured via GitHub Actions (`.github/workflows/deploy.yml`). The workflow uses `actions/upload-pages-artifact@v3` + `actions/deploy-pages@v4` — **do not add `actions/configure-pages`**, it returns 404 in this repo's setup and is not needed. The deployed site lives at `https://xuplus.github.io/Laundry-Alert/`.

The `github-pages` environment's OIDC trust was established manually via the GitHub web UI (Settings → Pages → GitHub Actions source). This is a one-time step and must not be undone.

## Architecture (`index.html`)

Everything is in a single file: HTML structure, CSS, and JavaScript.

**External APIs (no auth required):**
- [Open-Meteo](https://open-meteo.com/) — hourly forecast (temperature, humidity, precipitation probability, wind speed, cloud cover, weather code). `forecast_days=2&timezone=auto`.
- [Nominatim / OpenStreetMap](https://nominatim.org/) — forward geocoding (city search with suggestions) and reverse geocoding (lat/lon → place name).

**Key JS functions:**
- `checkLaundry()` — main entry point; fetches weather, runs verdict logic, renders result
- `estimateDryTime(temp, humidity, windspeed, cloudcover, method)` — returns estimated dry time in minutes based on conditions and drying method
- `getLocation()` — browser Geolocation API + Nominatim reverse geocode
- `fetchSuggestions()` / `showSuggestions()` / `selectPlace()` — debounced autocomplete for city search
- `applyLang()` — applies the active translation to all `data-i18n` elements and dynamic UI pieces

**Verdict logic** (in `checkLaundry()`):
1. Humidity > 85% and not indoor rack → "Not Recommended"
2. Temp < 5°C and not indoor rack → "Too Cold"
3. Rain (≥40% probability) arrives before wash + dry + 30 min buffer → "Don't Start!" or "Not Enough Time"
4. Rain arrives within wash + dry + 90 min → "Risky — Be Quick"
5. Otherwise → "Go For It!"

**i18n:**
- Language auto-detected from `navigator.language`; `es-*` → Spanish, everything else → English
- `TRANSLATIONS` object at top of `<script>` holds both `en` and `es` keys
- Static elements use `data-i18n="key"` attributes; `applyLang()` applies them on load
- Dynamic strings (verdict text, timeline labels, reason paragraphs) use `T.key` throughout `checkLaundry()` and other functions
- `document.title` and `html[lang]` are also set by `applyLang()`
