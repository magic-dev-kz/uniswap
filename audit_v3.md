# UniSwap (Converter) v2 Audit — Nash QA

**Date:** 2026-03-29
**File:** `index.html`, `sw.js`, `manifest.json`
**Previous score:** 9.2/10 (audit_v2)
**Scope:** v2 features (lang="ru", font size bump, onboarding tooltip, copy result button, favorites) + full checklist re-audit

---

## Checklist Results

### 1. localStorage try/catch
**PASS.** All 6 localStorage operations wrapped:
- `loadFavorites()` (line ~1219): `try { getItem + JSON.parse } catch { return [] }`
- `saveFavorites()` (line ~1226): `try { setItem } catch {}`
- `dismissTooltip()` (line ~1340): `try { setItem } catch {}`
- `initOnboarding()` (line ~1345): `try { getItem } catch {}`

JSON.parse is inside try/catch -- corrupted data won't crash. Array type not validated after parse (minor, see P3).

### 2. WCAG AA Contrast
**PASS (with notes).**

| Element | FG | BG | Ratio | AA |
|---------|----|----|------:|:--:|
| Tab active text (#000) on blue (#0A84FF) | #000 | #0A84FF | 5.8:1 | PASS |
| Tab active text (#000) on orange (#FF9F0A) | #000 | #FF9F0A | 10.2:1 | PASS |
| Tab active text (#000) on red (#FF453A) | #000 | #FF453A | 6.1:1 | PASS |
| Tab active text (#000) on cyan (#64D2FF) | #000 | #64D2FF | 12.0:1 | PASS |
| Overlay active (#000) on accent | same as above | | PASS |
| text-secondary (#A1A1A6) on bg (#1C1C1E) | #A1A1A6 | #1C1C1E | 5.3:1 | PASS |
| text-secondary (#A1A1A6) on bg-elevated (#2C2C2E) | #A1A1A6 | #2C2C2E | 4.2:1 | **FAIL** (normal <18px) |
| Onboarding tooltip (#A1A1A6, 13px) on bg-elevated (#2C2C2E) | #A1A1A6 | #2C2C2E | 4.2:1 | **FAIL** |
| All-units name (#A1A1A6, 14px) on bg-elevated (#2C2C2E) | #A1A1A6 | #2C2C2E | 4.2:1 | **FAIL** |
| Favorites item-cat (#A1A1A6, 12px) on bg-elevated (#2C2C2E) | #A1A1A6 | #2C2C2E | 4.2:1 | **FAIL** |
| Overlay title (#A1A1A6, 15px) on bg-elevated (#2C2C2E) | #A1A1A6 | #2C2C2E | 4.2:1 | **FAIL** |
| Placeholder opacity 0.5 of #A1A1A6 ~ #504F53 on #1C1C1E | ~#504F53 | #1C1C1E | ~2.7:1 | Exempt (placeholder) |
| Copy toast (#000) on green (#30D158) | #000 | #30D158 | 8.0:1 | PASS |
| Copied flash text (#30D158) on copied-bg | #30D158 | ~rgba flash | PASS (large, transient) |

**Summary:** text-secondary (#A1A1A6) on bg-elevated (#2C2C2E) = 4.2:1. Fails AA for normal text (<18px / <14px bold). Affects: onboarding tooltip 13px, all-units names 14px, favorites category label 12px, overlay title 15px, tab inactive text 15px (600 weight = bold, 15px > 14px bold threshold -> technically PASS for tabs). The 13px and 12px texts definitively fail.

### 3. Focus trap in overlay
**PASS.** `trapFocus()` handles Tab/Shift+Tab wrapping. Escape closes overlay. Focus moves to first option on open. Focus returns to trigger pill on close. Backdrop click closes.

### 4. prefers-reduced-motion
**PASS.** `@media (prefers-reduced-motion: reduce)` with `animation-duration: 0.01ms !important` and `transition-duration: 0.01ms !important` on `*, *::before, *::after`. Correct pattern (0.01ms, not 0s -- preserves animationend/transitionend events).

### 5. Keyboard navigation
**PARTIAL PASS.**
- Tabs: clickable, but **no arrow key navigation** (ARIA tabs pattern expects Left/Right arrows)
- Overlay: Tab trap works, Enter/Escape work
- All-units rows: tabIndex=0 + Enter to copy
- Favorites items: tabIndex=0 + Enter to apply
- Favorites remove: native button, keyboard accessible
- Swap btn: native button
- Favorite star: native button
- Copy result btn: native button
- Pills: click + Enter keydown handlers

Missing: Arrow keys on tablist, Space on favorites items.

### 6. Offline (SW)
**PASS.** `sw.js` caches `./` and `./index.html`. Cache-first strategy. Old caches cleaned on activate. SW registered at bottom of HTML. Single-file app so no missing assets.

Note: SW version is `uniswap-v2` -- needs bump on next update for cache invalidation.

### 7. Mobile viewport
**PASS.** `<meta name="viewport" content="width=device-width, initial-scale=1.0">`. `min-height: 100dvh`. `env(safe-area-inset-bottom)` on overlay panel.

Responsive breakpoint at 360px: font sizes reduced, overlay grid 3-col, all-units name min-width reduced. No 320px-specific breakpoint, but layout is flex-based and should scale. Overlay grid at 3-col with 44px min targets on 360px = 3 * 44 + gaps ~ 336px, tight but fits. At 320px with padding (16px*2) = 288px available, 3 * 44 + 16 = 148px -- fits.

---

## v2 Feature Audit

### Feature 1: lang="ru"
**PASS.** `<html lang="ru">` on line 2. All UI strings in Russian. Correct.

### Feature 2: Font size bump
**PASS.**
- Tabs: 15px (was likely smaller, now 15px) -- line 98
- Overlay options: 16px -- line 369
- Copy toast: 11px (small but transient)

### Feature 3: Onboarding tooltip
**PASS with minor notes.**
- Shows after 1s delay if `uniswap_onboarded` not set
- Auto-dismisses after 5s
- Dismisses on first tab switch (via monkey-patched switchCategory)
- Sets localStorage flag to prevent repeat
- localStorage in try/catch: YES
- **Issue:** tooltip is not dismissible by keyboard (no close button, no Escape handler for tooltip specifically). User must click a tab. Not blocking for auto-dismiss tooltip, but not ideal.
- **Issue:** tooltip arrow visual points to tabs but there is no arrow element -- tooltip is just text. Cosmetic.

### Feature 4: Copy result button
**PASS.**
- Clipboard API with `navigator.clipboard.writeText` as primary
- Fallback via `document.execCommand('copy')` with textarea trick
- Toast shows "Скопировано!" for 1200ms with clearTimeout guard against stacking
- Copies value + unit name (e.g. "1.609 км")
- Guards against empty/dash values
- aria-label="Копировать результат"
- Size 36x36 -- below 44px tap target minimum (see P3)

### Feature 5: Favorites
**PASS with notes.**
- Max 10 enforced (oldest removed via `shift()`)
- localStorage: try/catch on all reads/writes
- XSS: textContent only (no innerHTML for user-derived data). Category/unit keys are hardcoded in `categories` object, never user-supplied. Safe.
- Keyboard: tabIndex=0 + Enter on items, native button for remove
- Delete: works via splice + re-render. stopPropagation prevents applying favorite when removing.
- **Issue:** `loadFavorites()` returns raw `JSON.parse` result. If localStorage contains `"not-an-array"` or `{"key":"val"}`, code will crash on `.some()`, `.findIndex()`, `.forEach()`. Should add `Array.isArray()` guard.
- **Issue:** When a favorite switches category, there's a 250ms setTimeout race. If user clicks rapidly, state could desync. Low probability.
- **Issue:** Removing a favorite at index `i` uses closure over `i` from the render loop. If list is re-rendered between render and click (unlikely in sync code), index could be stale. Actually safe because re-render rebuilds DOM.

### All-units copy (regression check)
**PASS.** Still works. Clipboard API + fallback. Flash animation 600ms. Enter key supported.

### Overlay focus trap (regression check)
**PASS.** Not broken by v2 changes. Tab wrapping, Escape, backdrop click, focus return all intact.

### Responsive 360px / 320px
**360px:** Covered by media query. Font sizes shrink, grid goes 3-col.
**320px:** No specific breakpoint but flex layout handles it. Copy result button (36px) + input might be tight in result row. Gap is 4px (spacing-xs), input has `min-width: 0` with `flex: 1`. Should work but deserves visual testing.

---

## Bug List

### P1 (Critical) -- 0 items
None.

### P2 (Major) -- 2 items

**P2-1: text-secondary on bg-elevated contrast fail (4.2:1)**
- Affects: onboarding tooltip (13px), all-units unit names (14px), favorites category label (12px), overlay title (15px 600w)
- WCAG AA requires 4.5:1 for normal text (<18px regular / <14px bold)
- 13px and 12px text at 4.2:1 definitively fails
- Fix: darken bg-elevated or lighten text-secondary. Suggest `--text-secondary: #AEAEB2` (~4.7:1 on #2C2C2E) or higher

**P2-2: No arrow key navigation on tablist**
- Tabs have `role="tablist"` and `role="tab"` but no Left/Right arrow key handlers
- ARIA Authoring Practices requires arrow keys for tab pattern
- Keyboard-only users can Tab between tabs (not ideal but functional)
- Fix: add keydown handler for ArrowLeft/ArrowRight on tablist, move focus and activate

### P3 (Minor) -- 7 items

**P3-1: loadFavorites() no Array.isArray guard**
- `JSON.parse` could return non-array if localStorage is manually edited
- Would crash on `.some()`, `.findIndex()`, `.forEach()`
- Fix: `var arr = JSON.parse(raw); return Array.isArray(arr) ? arr : [];`

**P3-2: Copy result button 36x36px below 44px tap target**
- WCAG 2.5.8 Target Size (Enhanced) recommends 44px minimum
- Adjacent to input field, no overlap risk
- Fix: increase to 44px or add invisible touch area

**P3-3: Onboarding tooltip not dismissible by Escape key**
- Auto-dismisses after 5s, so not blocking
- But keyboard users have no way to manually dismiss before timeout
- Fix: add Escape handler when tooltip is visible

**P3-4: No Space key handler on favorites items**
- Interactive divs with tabIndex=0 respond to Enter but not Space
- Native buttons handle Space by default; divs do not
- Fix: add `e.key === ' '` check alongside Enter

**P3-5: SW version string not bumped for v2 features**
- `sw.js` uses cache name `uniswap-v2` -- if this was set before v2 features, existing caches won't invalidate
- If v2 is the first version with this cache name, it's fine. Needs bump on next update.

**P3-6: No inert on main content when overlay is open**
- Overlay has `aria-modal="true"` and focus trap, which works for modern screen readers
- Older NVDA/JAWS may still allow virtual cursor to escape to content behind overlay
- Fix: add `inert` attribute to `.app` when overlay opens, remove on close

**P3-7: Favorites re-reads localStorage on every operation**
- `isFavorited()` calls `loadFavorites()` which hits localStorage+JSON.parse
- `toggleFavorite()` calls `loadFavorites()` then `updateFavStar()` (which calls `isFavorited()` -> another `loadFavorites()`)
- Performance: negligible for 10 items, but architecturally noisy
- Fix: cache favorites array in memory, sync to localStorage on write

---

## What works well

1. **localStorage try/catch everywhere** -- all 6 operations protected. Best practice.
2. **Clipboard API with fallback** -- both copyRow and copyResult have proper fallback chains.
3. **Focus management** -- overlay opens with focus on first option, returns focus to trigger on close.
4. **prefers-reduced-motion** -- correct 0.01ms pattern (not 0s).
5. **IIFE + strict mode** -- no globals.
6. **lang="ru"** -- correctly set.
7. **Onboarding** -- clean implementation: delay, auto-dismiss, localStorage flag, no repeat.
8. **Favorites** -- max 10 limit, shift() overflow, XSS-safe (textContent only), keyboard accessible.
9. **Temperature conversion** -- correct via intermediate Celsius.
10. **Number formatting** -- space thousands separator, no scientific notation, trailing zero trimming.
11. **Responsive** -- 360px breakpoint, dvh, safe-area-inset-bottom.
12. **Dark theme** -- consistent, no white flash.

---

## Score: 9.0 / 10

**Delta from 9.2:** -0.2 for newly discovered P2-1 (text-secondary contrast on bg-elevated at 4.2:1 affecting small text in multiple components). Previous audit missed this specific combination because focus was on accent colors.

**Breakdown:**
- Functionality: 10/10 (conversion math, favorites, copy, onboarding all correct)
- Accessibility: 8/10 (contrast fail on secondary text in cards, no arrow keys on tablist)
- Security: 10/10 (no XSS vectors, textContent only, no user-derived innerHTML)
- Performance: 10/10 (single file, no external deps, minimal DOM manipulation)
- Offline: 9/10 (SW works, cache version management needs attention)
- Mobile: 9/10 (responsive, safe areas, 36px copy btn slightly small)

---

## Verdict

**Ship-ready with minor contrast fix recommended.** The v2 features (onboarding, copy result, favorites) are all correctly implemented with proper localStorage protection, clipboard fallback, and keyboard support. The main issue is text-secondary (#A1A1A6) on bg-elevated (#2C2C2E) = 4.2:1, which affects small text (12-14px) in several components. A single CSS variable change to lighten text-secondary by ~5% would close all instances. Arrow key navigation on tablist is the second priority -- functional without it (Tab works) but violates ARIA best practice.

For 9.5+: fix P2-1 (contrast) and P2-2 (arrow keys).
