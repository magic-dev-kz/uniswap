# UniSwap — Changelog

## v12.0 (2026-03-29) — "Power User"

Feature update by Mario.

### Conversion Cheat Sheet
- Collapsible "Common Conversions" section with 10 popular conversions (1 mile = 1.609 km, 1 lb = 0.454 kg, etc.)
- Click any entry to copy to clipboard with global toast confirmation
- Toggle via styled button with animated arrow

### Multi-step Conversion
- "Convert through" section: select an intermediate unit to see two-step conversion results
- Example: feet -> meters -> centimeters — shows both intermediate and final results
- Dropdown populated dynamically based on current category units (excludes from/to units)
- Hidden for currency category; requires 3+ units in category
- Updates live as input value changes

### Offline Currency Disclaimer
- When currency rates are loaded from cache (not fresh from API): orange warning banner
- Shows "Offline mode — using cached rates from [date]"
- Automatically hidden when fresh rates are fetched
- Styled with orange accent to match the stale-rates visual language

### Service Worker
- Cache version bumped to `uniswap-v12`

---

## v11.0 (2026-03-29) — "First Impression"

Feature update by Mario.

### Onboarding Overlay
- Full-screen glassmorphism overlay on first visit (localStorage-gated)
- Title with emoji, tagline "Convert anything. Instantly."
- Three feature bullets: categories + currency, math expressions, offline/no-tracking
- CTA button "Start Converting" dismisses overlay
- Dismiss via CTA click, backdrop click, or Escape key
- Blue-cyan gradient accent matching app theme
- Spring animation on card entrance

### Service Worker
- Cache version bumped to `uniswap-v11`

---

## v10.0 (2026-03-29) — "Micro-polish"

Feature update by Mario.

### Stale Rates Warning
- When cached currency rates are older than 24 hours, "Rates may be outdated" text turns yellow/orange

### Input Placeholder
- Conversion input already carries placeholder "e.g. 100"

### Tab Emoji Icons Larger
- Category tab emoji icons bumped from 1.1em to 1.5rem

### Service Worker
- Cache version bumped to `uniswap-v10`

---

## v9.0 (2026-03-29) — "Share, Polish & Icons"

Feature update by Mario.

### Share on Twitter/X
- Share button now opens Twitter/X with pre-filled tweet: "X km = Y miles" + link (replaces clipboard copy)

### Result Emphasis
- Result input font bumped from 28px to 36px (40px on desktop) with heavier weight
- Spring pop animation on every conversion result

### Category Tab Icons
- Emoji icons on each tab: Length, Weight, Temperature, Volume, Area, Currency

### Service Worker
- Cache version bumped to `uniswap-v9`

---

## v8.0 (2026-03-29) — "Area & Quick Access"

Feature update by Mario.

### New Category: Area (Площадь)
- 6th category tab with purple accent (`#BF5AF2`).
- 7 units: кв. метры (м²), кв. футы (фут²), кв. ярды (ярд²), акры, гектары (га), кв. километры (км²), кв. мили (миля²).
- Factor-based conversion through base unit (м²), same as length/weight/volume.
- Default pair: м² → фут².

### Quick Conversion Presets
- Row of preset buttons appears below the "from" unit pill.
- Each category has 3-4 common conversions (e.g. "1 mi → km", "1 lb → kg", "0°C → °F", "1 acre → м²", "100 USD → EUR").
- Clicking a preset fills in value, sets units, and triggers conversion immediately.
- Presets rebuild on category switch.

### Conversion Formula Display
- Below the result unit pill, a formula line shows the conversion ratio: "1 км = 0.621371 мили".
- For temperature, shows the actual formula (e.g. "°F = °C × 9/5 + 32").
- Hidden when from and to units are the same.
- Updates on every conversion/unit change.

### Technical
- Keyboard shortcuts expanded: `1`-`6` now switch categories.
- SW cache bumped to `uniswap-v8`.

### Preserved
- All existing features intact.
- HTML structure and accessibility attributes unchanged.

---

## v7.0 (2026-03-29) — "Deep Links & Memory"

Feature update by Mario.

### URL Hash State
- Current conversion is saved to the URL hash: `#100-km-mi`.
- On page load, the hash is parsed and the conversion is restored.
- Hash updates on every conversion change via `history.replaceState`.
- Browser back/forward triggers `hashchange` listener to restore state.
- Shareable links: copy the URL to share a specific conversion.

### Keyboard Shortcuts
- Keys `1`-`5` switch categories (Length, Weight, Temp, Volume, Currency) when not typing in an input field.
- `Enter` while focused on an input triggers conversion, records to history, and blurs the field.
- Shortcuts disabled when the unit overlay is open.

### Auto-Save Last Conversion
- Every conversion is saved to `localStorage` under `uniswap_last_conversion`.
- On page load (when no URL hash is present), the last conversion is restored: category, units, and input value.
- URL hash takes priority over saved state.

### Technical
- SW cache bumped to `uniswap-v7`.

### Preserved
- All existing features intact.
- HTML structure and accessibility attributes unchanged.

---

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
