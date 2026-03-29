# Converter App -- Design Specification
> Leo | 2026-03-29 | v1.0

## 1. Концепция

Конвертер единиц с zero-friction UX: ввёл число -- увидел результат во всех единицах категории мгновенно. iOS dark aesthetic, унифицированный с таймером (8.2/10). Отличие от конкурентов -- режим "все единицы" как основной, не как опция. Mobile-first, max-width 400px. Каждая категория имеет свой subtle accent color для визуального разделения контекстов.

**Референс:** iOS Calculator currency mode + Apple Settings unit picker, но быстрее.

---

## 2. Layout Structure

### Основной экран (длина, значение 100 км)

```
+------------------------------------------+
|  [Длина] [Вес] [Темп.] [Объём]           |  <- category tabs (pill style)
+------------------------------------------+
|                                           |
|             [ 100         ]               |  <- input (32px mono, bottom-line)
|             [ километры  v ]              |  <- from-unit pill selector
|                                           |
|               ( swap )                    |  <- 44x44 circle, arrows icon
|                                           |
|             = 62.1371                     |  <- result (28px mono, bold)
|             [ мили  v ]                   |  <- to-unit pill selector
|                                           |
+------ Все единицы ------------------------+
|  мм       100 000 000            [copy]   |
|  см       10 000 000             [copy]   |
|  м        100 000                [copy]   |
|  км       100                    [copy]   |
|  дюймы    3 937 008              [copy]   |
|  футы     328 084                [copy]   |
|  ярды     109 361                [copy]   |
|  мили     62.1371                [copy]   |
+------------------------------------------+
```

### Unit Selector (раскрытый)

```
+------------------------------------------+
|  Выберите единицу:                        |
|                                           |
|  [  мм  ] [  см  ] [   м  ] [  км  ]     |
|  [ дюймы] [ футы ] [ ярды ] [ мили ]     |
|                                           |
+------------------------------------------+
```

Overlay поверх контента. Полупрозрачный backdrop. Pill-кнопки в grid 4 колонки.

---

## 3. CSS Variables (Design Tokens)

```css
:root {
  /* === Base palette (inherited from timer) === */
  --bg:              #1C1C1E;
  --bg-elevated:     #2C2C2E;
  --bg-tertiary:     #3A3A3C;
  --text-primary:    #FFFFFF;
  --text-secondary:  #8E8E93;
  --blue:            #0A84FF;
  --green:           #30D158;
  --red:             #FF453A;
  --orange:          #FF9F0A;
  --gray:            #48484A;

  /* === Category accent colors === */
  --accent-length:   #0A84FF;   /* blue -- primary, default */
  --accent-weight:   #FF9F0A;   /* orange */
  --accent-temp:     #FF453A;   /* red */
  --accent-volume:   #64D2FF;   /* teal/cyan -- iOS system teal */

  /* === Category accent backgrounds (10% opacity for subtle tint) === */
  --accent-length-bg:  rgba(10, 132, 255, 0.10);
  --accent-weight-bg:  rgba(255, 159, 10, 0.10);
  --accent-temp-bg:    rgba(255, 69, 58, 0.10);
  --accent-volume-bg:  rgba(100, 210, 255, 0.10);

  /* === Typography === */
  --font-mono:       'SF Mono', 'Menlo', 'Consolas', monospace;
  --font-sans:       -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;

  /* === Sizing === */
  --app-max-width:   400px;
  --btn-height:      48px;
  --btn-radius:      24px;
  --tap-target:      44px;       /* minimum touch target per Apple HIG */
  --spacing-xs:      4px;
  --spacing-sm:      8px;
  --spacing-md:      16px;
  --spacing-lg:      24px;
  --spacing-xl:      32px;

  /* === Animation === */
  --ease-out:        cubic-bezier(0.25, 0.1, 0.25, 1);
  --ease-spring:     cubic-bezier(0.34, 1.56, 0.64, 1);  /* overshoot for swap */
  --duration-fast:   150ms;
  --duration-med:    300ms;
  --duration-slow:   500ms;

  /* === Dynamic (set by JS per category) === */
  --accent:          var(--accent-length);
  --accent-bg:       var(--accent-length-bg);
}
```

**Почему dynamic `--accent`:** JS меняет `--accent` и `--accent-bg` при смене категории. Все компоненты (input underline, active tab, swap button hover, result highlight) автоматически подхватывают текущий цвет категории через одну переменную. Ноль дублирования.

---

## 4. HTML Structure

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Converter</title>
  <!-- inline <style> + <script> as single-file app -->
</head>
<body>
  <div class="app">

    <!-- Category Tabs -->
    <div class="tabs" role="tablist">
      <button class="tab active" role="tab"
        data-category="length" aria-selected="true">Длина</button>
      <button class="tab" role="tab"
        data-category="weight" aria-selected="false">Вес</button>
      <button class="tab" role="tab"
        data-category="temp" aria-selected="false">Темп.</button>
      <button class="tab" role="tab"
        data-category="volume" aria-selected="false">Объём</button>
    </div>

    <!-- Converter Body -->
    <div class="converter" role="tabpanel">

      <!-- Input Section -->
      <div class="converter__input-section">
        <input
          type="text"
          inputmode="decimal"
          class="converter__input"
          id="value-input"
          value="1"
          autocomplete="off"
          autofocus>
        <button class="unit-pill" id="from-unit"
          aria-haspopup="listbox">
          <span class="unit-pill__label">километры</span>
          <span class="unit-pill__arrow">&#9662;</span>
        </button>
      </div>

      <!-- Swap Button -->
      <button class="swap-btn" id="swap-btn" aria-label="Поменять единицы местами">
        <svg class="swap-btn__icon" viewBox="0 0 24 24"
          width="20" height="20" fill="none"
          stroke="currentColor" stroke-width="2"
          stroke-linecap="round" stroke-linejoin="round">
          <path d="M7 4v16M7 4l-3 3M7 4l3 3" />
          <path d="M17 20V4M17 20l-3-3M17 20l3-3" />
        </svg>
      </button>

      <!-- Result Section -->
      <div class="converter__result-section">
        <div class="converter__result" id="result-value">= 0.621371</div>
        <button class="unit-pill" id="to-unit"
          aria-haspopup="listbox">
          <span class="unit-pill__label">мили</span>
          <span class="unit-pill__arrow">&#9662;</span>
        </button>
      </div>

      <!-- All Units List -->
      <div class="all-units" id="all-units">
        <div class="all-units__header">Все единицы</div>
        <!-- Dynamic rows: -->
        <!--
        <div class="all-units__row" data-unit="mm" tabindex="0">
          <span class="all-units__name">мм</span>
          <span class="all-units__value">1 000 000</span>
          <span class="all-units__copied">Copied</span>
        </div>
        -->
      </div>

    </div>

    <!-- Unit Selector Overlay -->
    <div class="unit-overlay" id="unit-overlay" hidden>
      <div class="unit-overlay__backdrop"></div>
      <div class="unit-overlay__panel">
        <div class="unit-overlay__title">Выберите единицу</div>
        <div class="unit-overlay__grid" id="unit-grid">
          <!-- Dynamic pills -->
        </div>
      </div>
    </div>

  </div>
</body>
</html>
```

### Ключевые решения по HTML

1. **`type="text"` + `inputmode="decimal"`** вместо `type="number"` -- позволяет свободное форматирование (пробелы-разделители, минус для температуры), при этом мобильная клавиатура показывает цифры с точкой.

2. **`role="tablist"` / `role="tab"`** -- accessibility, скринридеры понимают навигацию.

3. **`tabindex="0"` на строках all-units** -- навигация клавиатурой (Tab + Enter = copy).

4. **Overlay вместо dropdown** -- нативный `<select>` невозможно стилизовать на iOS. Кастомный overlay с pill-grid даёт полный контроль над визуалом и tap targets.

---

## 5. Component Specs

### 5.1 Category Tabs

```css
.tabs {
  display: flex;
  gap: var(--spacing-xs);
  background: var(--bg-elevated);
  border-radius: 10px;
  padding: 3px;
  margin-bottom: var(--spacing-xl);
}

.tab {
  flex: 1;
  padding: 10px 0;
  border: none;
  border-radius: 8px;
  background: transparent;
  color: var(--text-secondary);
  font-family: var(--font-sans);
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: all var(--duration-fast) var(--ease-out);
}

.tab.active {
  background: var(--accent);
  color: var(--text-primary);
}
```

**Поведение при переключении (двойной rAF из таймера):**

```
1. Убрать .active с текущего таба
2. requestAnimationFrame(() => {
     requestAnimationFrame(() => {
       Добавить .active на новый таб
       Обновить --accent, --accent-bg через JS
       Пересоздать unit pills и all-units список
     })
   })
```

Двойной rAF гарантирует, что браузер успевает отрисовать состояние "без активного таба" до появления нового. Без этого transition не срабатывает.

### 5.2 Input Field

```css
.converter__input {
  width: 100%;
  background: transparent;
  border: none;
  border-bottom: 2px solid var(--gray);
  color: var(--text-primary);
  font-family: var(--font-mono);
  font-size: 36px;
  font-weight: 300;
  font-variant-numeric: tabular-nums;
  text-align: center;
  padding: var(--spacing-sm) 0;
  outline: none;
  caret-color: var(--accent);
  transition: border-color var(--duration-fast) var(--ease-out);
}

.converter__input:focus {
  border-bottom-color: var(--accent);
}

.converter__input::placeholder {
  color: var(--text-secondary);
  opacity: 0.5;
}
```

| Property | Value | Why |
|---|---|---|
| font-size | 36px | Крупно, читаемо с расстояния вытянутой руки |
| font-weight | 300 (light) | Визуально легче, не давит. Как у Apple Calculator |
| tabular-nums | always | Цифры не прыгают при вводе |
| border-bottom only | 2px | Минимальный хром, фокус на числе |
| caret-color | var(--accent) | Курсор ввода в цвете категории |

### 5.3 Unit Pill Selector

```css
.unit-pill {
  display: inline-flex;
  align-items: center;
  gap: var(--spacing-sm);
  height: var(--tap-target);            /* 44px -- Apple HIG minimum */
  padding: 0 var(--spacing-md);
  background: var(--bg-elevated);
  border: 1.5px solid var(--gray);
  border-radius: 22px;                  /* fully rounded */
  color: var(--text-primary);
  font-family: var(--font-sans);
  font-size: 15px;
  font-weight: 500;
  cursor: pointer;
  transition: border-color var(--duration-fast) var(--ease-out),
              background var(--duration-fast) var(--ease-out);
}

.unit-pill:hover,
.unit-pill:focus-visible {
  border-color: var(--accent);
  background: var(--accent-bg);
}

.unit-pill__arrow {
  font-size: 10px;
  color: var(--text-secondary);
  transition: transform var(--duration-fast) var(--ease-out);
}

.unit-pill[aria-expanded="true"] .unit-pill__arrow {
  transform: rotate(180deg);
}
```

### 5.4 Swap Button

```css
.swap-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: var(--tap-target);             /* 44px */
  height: var(--tap-target);
  border-radius: 50%;
  border: 1.5px solid var(--gray);
  background: var(--bg-elevated);
  color: var(--text-secondary);
  cursor: pointer;
  margin: var(--spacing-md) auto;
  transition: transform var(--duration-med) var(--ease-spring),
              border-color var(--duration-fast) var(--ease-out),
              color var(--duration-fast) var(--ease-out);
}

.swap-btn:hover,
.swap-btn:focus-visible {
  border-color: var(--accent);
  color: var(--accent);
}

.swap-btn:active {
  transform: scale(0.9);
}

/* Rotation animation on click (triggered by JS) */
.swap-btn--rotating {
  transform: rotate(180deg);
  transition: transform var(--duration-med) var(--ease-spring);
}
```

**Анимация swap (JS):**

```
1. Добавить класс .swap-btn--rotating
2. Поменять from/to единицы
3. Пересчитать результат
4. После transitionend -- убрать класс, сбросить rotation через
   requestAnimationFrame (чтобы следующий клик снова анимировался)
```

Easing `ease-spring` (cubic-bezier с overshoot 1.56) даёт ощущение "физической" кнопки -- стрелки чуть проскакивают за 180 градусов и возвращаются.

### 5.5 Result Display

```css
.converter__result {
  font-family: var(--font-mono);
  font-size: 28px;
  font-weight: 400;
  font-variant-numeric: tabular-nums;
  color: var(--accent);
  text-align: center;
  padding: var(--spacing-sm) 0;
  transition: color var(--duration-med) var(--ease-out);
}
```

Результат окрашен в accent цвет категории -- визуально выделяет ответ от ввода. Контрасты:

| Accent | on #1C1C1E | WCAG |
|---|---|---|
| #0A84FF (blue) | 4.6:1 | AA |
| #FF9F0A (orange) | 7.6:1 | AAA |
| #FF453A (red) | 5.3:1 | AA |
| #64D2FF (teal) | 9.8:1 | AAA |

### 5.6 All Units List

```css
.all-units {
  margin-top: var(--spacing-lg);
  border-top: 1px solid var(--bg-tertiary);
  padding-top: var(--spacing-md);
}

.all-units__header {
  font-family: var(--font-sans);
  font-size: 13px;
  font-weight: 600;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: var(--spacing-sm);
}

.all-units__row {
  display: flex;
  align-items: center;
  padding: 10px var(--spacing-md);
  border-radius: 10px;
  cursor: pointer;
  transition: background var(--duration-fast) var(--ease-out);
  position: relative;           /* for .all-units__copied positioning */
}

.all-units__row:hover,
.all-units__row:focus-visible {
  background: var(--bg-elevated);
}

.all-units__name {
  font-family: var(--font-sans);
  font-size: 14px;
  color: var(--text-secondary);
  min-width: 60px;
  flex-shrink: 0;
}

.all-units__value {
  font-family: var(--font-mono);
  font-size: 15px;
  font-variant-numeric: tabular-nums;
  color: var(--text-primary);
  flex: 1;
  text-align: right;
}

/* Copy feedback */
.all-units__row--copied {
  background: rgba(48, 209, 88, 0.15);     /* green flash */
}

.all-units__copied {
  position: absolute;
  right: var(--spacing-md);
  font-family: var(--font-sans);
  font-size: 12px;
  font-weight: 600;
  color: var(--green);
  opacity: 0;
  transform: translateY(4px);
  transition: opacity var(--duration-fast) var(--ease-out),
              transform var(--duration-fast) var(--ease-out);
}

.all-units__row--copied .all-units__copied {
  opacity: 1;
  transform: translateY(0);
}

.all-units__row--copied .all-units__value {
  opacity: 0;
}
```

**Copy feedback sequence (JS):**

```
1. Click/Enter on row
2. navigator.clipboard.writeText(value)
3. Add .all-units__row--copied
4. setTimeout 600ms -> remove .all-units__row--copied
```

Зелёный flash (#30D158 at 15% opacity) -- визуально однозначен "скопировано", не нужен toast/snackbar.

### 5.7 Unit Overlay

```css
.unit-overlay {
  position: fixed;
  inset: 0;
  z-index: 100;
  display: flex;
  align-items: flex-end;        /* slide up from bottom, like iOS action sheet */
  justify-content: center;
}

.unit-overlay[hidden] {
  display: none;
}

.unit-overlay__backdrop {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  animation: fade-in var(--duration-fast) var(--ease-out);
}

.unit-overlay__panel {
  position: relative;
  width: 100%;
  max-width: var(--app-max-width);
  background: var(--bg-elevated);
  border-radius: 16px 16px 0 0;
  padding: var(--spacing-lg);
  animation: slide-up var(--duration-med) var(--ease-out);
}

.unit-overlay__title {
  font-family: var(--font-sans);
  font-size: 15px;
  font-weight: 600;
  color: var(--text-secondary);
  margin-bottom: var(--spacing-md);
  text-align: center;
}

.unit-overlay__grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: var(--spacing-sm);
}

.unit-overlay__option {
  height: var(--tap-target);
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  border: 1.5px solid var(--gray);
  background: transparent;
  color: var(--text-primary);
  font-family: var(--font-sans);
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all var(--duration-fast) var(--ease-out);
}

.unit-overlay__option:hover,
.unit-overlay__option:focus-visible {
  border-color: var(--accent);
  background: var(--accent-bg);
}

.unit-overlay__option--active {
  background: var(--accent);
  border-color: var(--accent);
  color: #FFFFFF;
  font-weight: 600;
}

@keyframes slide-up {
  from { transform: translateY(100%); }
  to   { transform: translateY(0); }
}

@keyframes fade-in {
  from { opacity: 0; }
  to   { opacity: 1; }
}
```

Overlay появляется снизу (iOS action sheet паттерн). Закрывается по тапу на backdrop или по выбору единицы.

---

## 6. Animations Summary

| Animation | Property | Duration | Easing | Trigger |
|---|---|---|---|---|
| Tab switch | background, color | 150ms | ease-out | Tab click (double rAF) |
| Category transition | opacity, translateY | 300ms | ease-out | Category change |
| Input focus | border-color | 150ms | ease-out | Focus/blur |
| Swap rotation | transform: rotate(180deg) | 300ms | ease-spring (overshoot) | Swap click |
| Swap press | transform: scale(0.9) | 150ms | ease-out | :active |
| Copy flash | background (green 15%) | 600ms total | ease-out in, instant out | Row click |
| Copy text swap | opacity, translateY | 150ms | ease-out | Row click |
| Overlay backdrop | opacity 0->1 | 150ms | ease-out | Overlay open |
| Overlay panel | translateY 100%->0 | 300ms | ease-out | Overlay open |
| Unit pill hover | border-color, background | 150ms | ease-out | Hover/focus |
| Result value update | opacity flash | 200ms | ease-out | Input change |

### Result value update animation

При вводе числа результат не просто меняется, а кратко "мигает" -- opacity 1 -> 0.6 -> 1 за 200ms. Это даёт тактильную обратную связь "я пересчитал".

```css
.converter__result--flash {
  animation: value-flash 200ms var(--ease-out);
}

@keyframes value-flash {
  0%   { opacity: 1; }
  30%  { opacity: 0.6; }
  100% { opacity: 1; }
}
```

---

## 7. Color Palette by Category

Каждая категория имеет свой accent color. Он проявляется в:
- Активном табе (fill)
- Подчеркивании input при фокусе
- Цвете результата
- Border unit pill при hover
- Background unit overlay option при hover

| Category | Accent | Hex | Semantic | Contrast on #1C1C1E |
|---|---|---|---|---|
| Длина | Blue | #0A84FF | Нейтральный, "по умолчанию" | 4.6:1 (AA) |
| Вес | Orange | #FF9F0A | Физическая масса, "тяжесть" | 7.6:1 (AAA) |
| Температура | Red | #FF453A | Жар, тепло | 5.3:1 (AA) |
| Объём | Teal | #64D2FF | Вода, жидкость | 9.8:1 (AAA) |

Все цвета из стандартной палитры iOS 18. Контраст >= 4.5:1 (WCAG AA) гарантирован.

---

## 8. Typography Scale

| Element | Font | Size | Weight | Features | Purpose |
|---|---|---|---|---|---|
| Input value | SF Mono | 36px | 300 | tabular-nums | Главный фокус -- число |
| Result value | SF Mono | 28px | 400 | tabular-nums | Ответ, чуть мельче ввода |
| All-units value | SF Mono | 15px | 400 | tabular-nums | Компактный список |
| Tab label | System sans | 13px | 600 | -- | Навигация |
| Unit pill label | System sans | 15px | 500 | -- | Читаемость на pill |
| Section header | System sans | 13px | 600 | uppercase, letter-spacing 0.5px | "Все единицы" |
| Unit name (list) | System sans | 14px | 400 | -- | Вторичный контекст |
| Overlay option | System sans | 14px | 500 | -- | Выбор единицы |

**Иерархия по размеру:** 36 -> 28 -> 15 -> 14 -> 13. Три чётких уровня: ввод, результат, детали.

---

## 9. Responsive Breakpoints

```css
/* === Default: mobile-first, any width === */
.app {
  width: 100%;
  max-width: var(--app-max-width);   /* 400px */
  padding: var(--spacing-md);        /* 16px */
}

body {
  display: flex;
  justify-content: center;
  min-height: 100dvh;
  padding: var(--spacing-md);
}

/* === Extra-small (< 360px): compact mode === */
@media (max-width: 360px) {
  .converter__input {
    font-size: 28px;
  }
  .converter__result {
    font-size: 22px;
  }
  .tab {
    font-size: 12px;
    padding: 8px 0;
  }
  .unit-overlay__grid {
    grid-template-columns: repeat(3, 1fr);   /* 3 col on narrow screens */
  }
  .all-units__name {
    min-width: 48px;
  }
}

/* === Small (360-400px): default, no changes needed === */

/* === Tablet+ (>= 600px): more breathing room === */
@media (min-width: 600px) {
  body {
    padding: var(--spacing-xl);
  }
  .converter__input {
    font-size: 40px;
  }
  .converter__result {
    font-size: 32px;
  }
}
```

---

## 10. Spacing & Layout Rules

```
+------------------  app  ------------------+
|  padding: 16px                            |
|                                           |
|  [tabs]  margin-bottom: 32px              |
|                                           |
|  [input section]                          |
|    input: full width, center-aligned      |
|    unit pill: centered below, mt 12px     |
|                                           |
|  [swap] margin: 16px auto (centered)      |
|                                           |
|  [result section]                         |
|    result: full width, center-aligned     |
|    unit pill: centered below, mt 12px     |
|                                           |
|  [all-units] margin-top: 24px             |
|    border-top: 1px #3A3A3C                |
|    padding-top: 16px                      |
|    rows: 10px vertical padding each       |
|                                           |
+-------------------------------------------+
```

Весь converter body -- flex column, align-items center. Input и result sections также flex column, centered.

---

## 11. Interaction States

### Input Field

| State | Border-bottom | Caret |
|---|---|---|
| Default | 2px var(--gray) #48484A | hidden |
| Focused | 2px var(--accent) | var(--accent) |
| Error (non-numeric) | 2px var(--red) | var(--red) |
| Empty | 2px var(--gray), placeholder "0" | hidden |

### Unit Pill

| State | Border | Background |
|---|---|---|
| Default | 1.5px var(--gray) | var(--bg-elevated) |
| Hover / Focus | 1.5px var(--accent) | var(--accent-bg) |
| Active (pressed) | 1.5px var(--accent) | var(--accent-bg), scale(0.97) |
| Overlay open | 1.5px var(--accent) | var(--accent-bg), arrow rotated |

### Swap Button

| State | Border | Color | Transform |
|---|---|---|---|
| Default | 1.5px var(--gray) | var(--text-secondary) | none |
| Hover | 1.5px var(--accent) | var(--accent) | none |
| Active | 1.5px var(--accent) | var(--accent) | scale(0.9) |
| Animating | 1.5px var(--accent) | var(--accent) | rotate(180deg) |

### All-Units Row

| State | Background | Value text |
|---|---|---|
| Default | transparent | var(--text-primary) |
| Hover | var(--bg-elevated) | var(--text-primary) |
| Copied (0-600ms) | rgba(48,209,88,0.15) | hidden, replaced by "Copied" in green |

---

## 12. Accessibility Spec

| Requirement | Implementation |
|---|---|
| Keyboard nav | Tab through: tabs -> input -> from pill -> swap -> to pill -> all-units rows |
| Enter on unit pill | Opens overlay |
| Escape in overlay | Closes overlay, returns focus to pill |
| Enter on all-units row | Copies value |
| Screen reader | aria-label on swap, aria-haspopup on pills, role=tablist/tab on tabs |
| Focus visible | 2px outline var(--accent), offset 2px (on all interactive elements) |
| Color alone | Never the sole indicator. Copied state: color + text change + background |
| Contrast | All text meets WCAG AA 4.5:1 minimum |
| autofocus | Input field gets focus on page load |

```css
:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}
```

---

## 13. Number Formatting Rules

| Rule | Example | Rationale |
|---|---|---|
| Max 6 significant digits | 62.1371, not 62.137119... | Достаточно для бытовых конвертаций |
| Remove trailing zeros | 100, not 100.000000 | Чистота |
| Space as thousands separator | 1 000 000, not 1000000 | Читаемость (ГОСТ 8.417) |
| Negative allowed | -40 (temperature) | Celsius/Fahrenheit |
| Decimal point, not comma | 3.14, not 3,14 | Единообразие (P1: comma support) |
| Zero input | Shows 0 for all units | Не ошибка |
| Empty input | Treated as 0 | Graceful fallback |
| NaN/Infinity | Input field turns red, result shows "--" | Edge case для деления на ноль (не применимо к MVP, но заложить) |

---

## 14. File Structure

```
/Volumes/OpenClaw/sandbox/projects/converter/
  research.md       <- market research (exists)
  spec.md           <- product spec (exists)
  design.md         <- this file
  index.html        <- single-file app (CSS + JS inline)
```

---

## 15. Implementation Notes

1. **Single HTML file** -- CSS в `<style>`, JS в `<script>`, zero dependencies. Должен весить < 30KB.

2. **`inputmode="decimal"`** -- вызывает цифровую клавиатуру с точкой на мобильных. `type="text"` позволяет свободный парсинг (минус, пробелы).

3. **Conversion data structure** -- объект с коэффициентами к базовой единице. Температура -- отдельные формулы (не линейная зависимость).

4. **`font-variant-numeric: tabular-nums`** -- обязателен на всех числовых элементах. Без него цифры "прыгают" при реалтайм-обновлении (разная ширина 1 vs 8).

5. **Double rAF pattern** для tab transitions -- из таймера, проверенный. Без него CSS transitions не срабатывают при мгновенной смене классов.

6. **`100dvh`** вместо `100vh` -- корректная высота на мобильных Safari с динамической адресной строкой.

7. **Clipboard API** -- `navigator.clipboard.writeText()`. Fallback для старых браузеров: `document.execCommand('copy')` с временным `<textarea>`.

8. **Dynamic `--accent`** -- JS при смене категории делает `document.documentElement.style.setProperty('--accent', newColor)`. Все компоненты автоматически подхватывают.

9. **Overlay закрытие** -- три способа: клик по backdrop, клик по опции, Escape. Все три обязательны.

10. **Swap animation reset** -- после rotate(180deg) нужен сброс через rAF без transition, иначе второй клик анимирует обратно (а нужно снова вперёд).
