# UniSwap (Converter) -- Fresh Audit by Nash

**Дата:** 2026-03-29 (ночь)
**Предыдущая оценка:** 8.8/10
**Текущая оценка: 8.8/10** (подтверждена, не завышена)
**Цель:** 9.0+

---

## Чеклист

| # | Пункт | Статус |
|---|-------|--------|
| 1 | localStorage try/catch | N/A (не используется) |
| 2 | WCAG AA контраст 4.5:1 | **FAIL** -- 3 P2 |
| 3 | Focus trap в модалках | PASS |
| 4 | prefers-reduced-motion | **FAIL** -- P3 |
| 5 | Keyboard navigation | PASS (частично -- нет arrow keys для табов) |
| 6 | Offline (PWA) | N/A (single-file, работает из файла) |
| 7 | Mobile viewport | PASS |
| 8 | escapeHtml (" и ') | N/A (innerHTML только с hardcoded data) |
| 9 | Конверсия единиц -- математика | PASS |
| 10 | Temperature формулы | PASS |

---

## Что блокирует 9.0+

### P2 -- MUST FIX (3 бага)

**P2-1: Active tab white text на цветных фонах проваливает WCAG AA**

Активный таб использует `background: var(--accent)` + `color: var(--text-primary)` (#FFFFFF).

| Категория | Accent цвет | Контраст white | Вердикт |
|-----------|-------------|----------------|---------|
| Длина | #0A84FF (blue) | 3.65:1 | FAIL (13px = normal text, нужен 4.5:1) |
| Вес | #FF9F0A (orange) | 2.06:1 | **CRITICAL FAIL** |
| Темп. | #FF453A (red) | 3.41:1 | FAIL |
| Объём | #64D2FF (cyan) | 1.72:1 | **CRITICAL FAIL** |

**Фикс:** Для табов использовать тёмный текст на светлых акцентах ИЛИ затемнить акценты ИЛИ использовать `color: var(--bg)` на активном табе. Альтернатива: не заливать таб цветом, а использовать border-bottom + цветной текст.

**P2-2: unit-overlay__option--active -- та же проблема**

`.unit-overlay__option--active` использует `background: var(--accent)` + `color: #FFFFFF`. Те же контрастные провалы для orange/cyan/red.

**Фикс:** Аналогично P2-1.

**P2-3: .all-units__row hover не работает**

`.all-units__row:hover` устанавливает `background: var(--bg-elevated)`, но родитель `.all-units` уже имеет `background: var(--bg-elevated)`. Hover визуально НЕ ОТЛИЧИМ.

**Фикс:** Изменить hover background на `var(--bg-tertiary)` или `var(--accent-bg)`.

### P3 -- SHOULD FIX (2 бага)

**P3-1: Нет prefers-reduced-motion**

5 анимаций без respect к пользовательским настройкам:
- fade-in (backdrop)
- slide-up (overlay panel)
- value-flash (result input)
- converter--fading (category switch)
- swap-btn--rotating (swap button)

**Фикс:**
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

**P3-2: Tab arrow key navigation**

ARIA tab pattern рекомендует навигацию стрелками влево/вправо между табами. Сейчас только клик и Tab.

**Фикс:** Добавить keydown handler для ArrowLeft/ArrowRight на tablist.

### P4 -- NICE TO HAVE (1 баг)

**P4-1: parseInput заменяет только первую запятую**

`.replace(',', '.')` заменяет только первое вхождение. Ввод `1,234,567` станет `1.234,567` = NaN.

**Фикс:** `.replace(/,/g, '.')` -- но тогда `1.234.567` = 1.234. Лучше: если запятая одна -- заменить на точку, если несколько -- удалить (тысячные разделители).

---

## Минимум фиксов для 9.0+

| # | Фикс | Влияние |
|---|------|---------|
| 1 | Контраст active tabs/overlay options | +0.15 (3 P2 -> 0 P2) |
| 2 | prefers-reduced-motion | +0.05 |
| 3 | all-units__row hover bg | включён в P2 |

**Итого 3 фикса для перехода порога 9.0.**

Математика, температура, focus trap, keyboard, viewport -- всё солидно. Единственный серьёзный блокер -- контраст цветных акцентов с белым текстом. Это системная проблема: 3 из 4 категорий проваливают AA на активных элементах.

---

## Итоговая оценка

- **Функциональность:** 10/10 (все конверсии верны)
- **Accessibility:** 7.5/10 (контраст tabs + no reduced-motion)
- **UX:** 9/10 (hover bug, но в целом отлично)
- **Security:** 10/10 (нет user input в innerHTML)
- **Code quality:** 9/10 (чистый, readable)

**Общая: 8.8/10** -- подтверждена. Для 9.0+ нужны P2-1 + P2-2 + P2-3 + P3-1.
