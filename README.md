# 🔄 UniSwap — Universal Unit Converter

**Convert anything. Instantly. Offline.**

A fast, beautiful unit converter with dynamic accent colors per category. Pin your favorite conversions, copy results, works offline.

## Features

- **4 categories** — Length, Weight, Temperature, Volume
- **Dynamic accent colors** — each category has its own color identity
- **Favorites** — pin up to 10 conversion pairs for quick access
- **Copy result** — one-tap clipboard copy with unit name
- **All units view** — see every conversion at once
- **Bidirectional** — edit either field, result updates instantly
- **Works offline** — PWA, zero dependencies
- **Russian UI** — built for Russian-speaking users

## How it works

1. Pick a category (tabs with arrow key navigation)
2. Enter a value
3. Pick units (overlay selector)
4. See result instantly
5. Star to save as favorite

## Tech

- Single HTML file (~30KB)
- Vanilla JS, zero frameworks
- Factor-based conversion (temperature via Celsius intermediate)
- Overlay unit selector with focus trap
- ARIA tabs pattern with arrow keys
- PWA with service worker

## License

MIT
