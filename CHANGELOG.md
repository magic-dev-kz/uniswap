# UniSwap — Changelog

## v6.0 (2026-03-29) — "Polish & Motion"

Quality polish by Mario.

### Result CountUp Animation
- Conversion result animates from 0 to the final value over 0.5s with ease-out cubic easing.
- Animation skipped during continuous typing for responsive feel.
- Respects `prefers-reduced-motion`.

### Currency Rates Toast
- When fresh rates are fetched from the API, a "Rates updated" toast slides in at the top of the screen.
- Auto-dismisses after 2 seconds with smooth spring transition.

### Swap Animation Enhancement
- When pressing the swap button, input and result sections cross-fade with vertical slide.
- Sections fade out in opposite directions, values swap, then sections fade back in from swapped positions.
- Smooth spring easing for a polished feel.

### Focus-Visible Audit
- All interactive elements verified for `:focus-visible` outlines.
- Global rule covers tabs, pills, overlay options, all-units rows, copy/share/star buttons, favorites, and history items.
- Inputs use border-bottom glow instead of outline (intentional).

### Technical
- SW cache bumped to `uniswap-v6`.

### Preserved
- All existing features intact.
- HTML structure and accessibility attributes unchanged.

---

## v5.0 (2026-03-29) — "World Money & Smart Input"

### Added
- **Live Currency Conversion** — 5th category "Валюта" with 10 popular currencies (USD, EUR, GBP, JPY, CNY, RUB, KZT, TRY, INR, BRL)
  - Free API: open.er-api.com (no key required)
  - Rates cached in localStorage, refreshed every hour
  - Offline fallback: uses cached rates when network unavailable
  - Status indicator with green/orange dot and last-update date
- **Quick Calculator** — input field accepts math expressions (`2+3`, `10*5.5`, `100/3`, parentheses)
  - Safe recursive-descent parser (no `eval()`)
  - Inline hint shows computed value below input (e.g. "= 13")
- **Share Conversion** — new share button next to copy
  - Copies full result with attribution: "100 km = 62.14 miles — via UniSwap"
  - Global toast notification "Copied!"

### Changed
- SW cache bumped to `uniswap-v5`
- Service worker passes currency API requests to network-first strategy

### Preserved
- All existing features (favorites, history, onboarding, copy, swap animation, all-units, bidirectional input)
- HTML structure and accessibility attributes intact

## v4.0 (2026-03-29) — "Memory Lane"

### Added
- **History** — last 10 conversions stored in localStorage, displayed as a dropdown list below favorites
  - Each entry shows full conversion (e.g. "100 км = 62.1371 мили")
  - Click any entry to restore category, units, and values
  - "Очистить" button clears all history
  - Duplicate consecutive entries are suppressed
  - History is recorded on input blur, unit change, and swap
- **Improved Copy Result** — clipboard now copies full equation format: "100 км = 62.1371 мили" (was: value + unit only)

### Changed
- Swap button event binding uses indirect call for proper override support
- SW cache bumped to `uniswap-v4`

### Preserved
- All existing features (favorites, onboarding, copy, swap animation, all-units, bidirectional input)
- HTML structure and accessibility attributes intact

## v3.0 (2026-03-29) — "Glass & Glow"
### Design overhaul (CSS-only, zero JS changes)

#### Glassmorphism
- All cards, panels, tabs, pills, and overlays now use semi-transparent backgrounds with `backdrop-filter: blur()` for a frosted-glass aesthetic
- New tokens: `--glass-border` (`rgba(255,255,255,0.08)`), `--glass-blur` (`20px`)

#### Modern Gradients
- Each category has a dual-color gradient pair (`--accent-*` / `--accent-*-end`):
  - Length: blue -> cyan
  - Weight: orange -> gold
  - Temperature: red -> pink
  - Volume: cyan -> indigo
- Active tab, active overlay option, and result text render with `linear-gradient(135deg)`
- Body background uses subtle radial gradients for depth

#### Google Fonts (Inter)
- Connected Inter (300-700) via Google Fonts CDN with `preconnect`
- `--font-sans` defaults to Inter with system-font fallbacks

#### Micro-animations
- Tabs: hover color lift, active scale-down, gradient glow on selection
- Unit pills: hover lifts (`translateY(-1px)`) with colored box-shadow
- Swap button: smooth 180deg spring rotation; glowing `::before` halo on hover
- Input focus: colored glow shadow under the active input line
- All-units rows: `translateX(2px)` slide on hover
- Favorites: slide-right on hover with shadow
- Copy / star buttons: scale up/down on hover/active

#### Colored Glow Shadows
- New `--accent-glow-*` tokens per category
- Applied to active tabs, swap button, pills, copy button, star button

#### Swap Button
- Enlarged to 52px for better tap target
- `::before` pseudo-element creates a blurred gradient halo on hover
- Rotation uses `--duration-slow` for smoother feel

#### Mobile Polish
- New `@media (max-width: 400px)` breakpoint for tab font scaling
- Swap button shrinks on very small screens (360px)
- Overlay panel uses 40px blur with saturation boost

#### Dark Theme
- Darker base (`#0F0F11`) for richer contrast
- Elevated surfaces: `rgba(255,255,255,0.06)` for true glassmorphism
- Borders: `rgba(255,255,255,0.08)` for subtle separation

### Preserved
- All JavaScript logic untouched
- All `id`, `data-*`, and `aria-*` attributes intact
- HTML structure identical to v2

## v2.0 (2026-03-29)
### Added
- **Favorites** — pin up to 10 conversion pairs, stored in localStorage
- **Copy Result** — clipboard button with toast feedback ("Copied!" / "Скопировано!")
- **Onboarding tooltip** — first-run hint about category tabs, auto-dismiss 5s
- **Arrow key navigation** on tablist (ARIA tabs pattern)

### Fixed
- `lang="ru"` attribute (UI is in Russian)
- Font size bump: tabs 13px->15px, overlay options 14px->16px
- `--text-secondary` contrast #A1A1A6->#AEAEB2 (4.7:1 on bg-elevated)
- Copy button tap target increased to 44px
- `Array.isArray()` guard in favorites loader
- Escape key dismisses onboarding tooltip

### Changed
- SW cache bumped to `uniswap-v3`

## v1.0 (2026-03-29)
- Initial release
- 4 categories: Length, Weight, Temperature, Volume
- Dynamic accent color per category
- All units list with click-to-copy
- Overlay unit selector with focus trap
- Bidirectional conversion
- PWA (service worker + manifest)
