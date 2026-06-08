# Laundry Alert

A single-page web app that tells you whether it's a good time to do laundry based on your local weather forecast.

**[Try it → xuplus.github.io/Laundry-Alert](https://xuplus.github.io/Laundry-Alert/)**

## How it works

Enter your city (or share your location), pick a drying method and wash cycle length, and the app checks the next 48 hours of weather to estimate how long your laundry will take to dry and whether rain will arrive before it's done.

Drying time is estimated from temperature, humidity, wind speed, and cloud cover using a physics-based model derived from Dalton's evaporation law — the same framework used by meteorological services for "washing day" forecasts. See [docs/drying-science.md](docs/drying-science.md) for the full explanation, formula, and sources.

## Data sources

- **Weather:** [Open-Meteo](https://open-meteo.com/) — free, no API key required
- **Geocoding:** [Nominatim / OpenStreetMap](https://nominatim.org/)

## Development

No build step. Edit `index.html` and open it in a browser. Push to `main` to deploy via GitHub Pages.
