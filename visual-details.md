# Visual Details

Hierarchy, cursor affordance, and icon weight.

## De-emphasize to Emphasize

A common instinct when building hierarchy is to make the primary element louder — bigger, bolder, higher contrast. This works, but it has a ceiling. Every element can only get so large or so bold before it becomes noise.

The more effective approach is the inverse: reduce the prominence of secondary elements. Lower their contrast, reduce their weight, decrease their size. The primary element reads as more important not because it changed, but because the contrast differential increased.

### In practice

```css
/* Instead of making the heading bigger */
.heading { font-size: 2rem; font-weight: 700; color: var(--color-foreground); }

/* Reduce the supporting text */
.subheading {
  font-size: 0.875rem;
  font-weight: 400;
  color: var(--color-muted-foreground); /* lower contrast */
}
```

Apply this to every layer of a component: primary action vs secondary action, main content vs metadata, active tab vs inactive tabs. The hierarchy becomes self-evident when secondary elements step back rather than when primary elements step forward.

This principle also applies to layout: dense components with many equal-weight elements compete for attention. Reduce the visual weight of non-essential UI (labels, icons, borders, dividers) and the important content emerges naturally.

## Cursor Affordance

The cursor is a free, zero-cost affordance signal. It communicates intent before the user commits to an interaction. Mismatching the cursor to the element's behavior creates a moment of uncertainty that erodes trust in the interface.

| Interaction type | Cursor value |
| --- | --- |
| Clickable link or button | `pointer` |
| Editable text field | `text` |
| Draggable element (idle) | `grab` |
| Draggable element (active) | `grabbing` |
| Disabled element | `not-allowed` |
| Horizontal resize handle | `ew-resize` |
| Vertical resize handle | `ns-resize` |
| Corner resize | `nwse-resize` / `nesw-resize` |
| Crosshair / precision target | `crosshair` |
| Loading / processing | `wait` |
| Help / tooltip trigger | `help` |

```css
.button     { cursor: pointer; }
.drag-item  { cursor: grab; }
.drag-item:active { cursor: grabbing; }
.disabled   { cursor: not-allowed; pointer-events: none; }
.resize-col { cursor: ew-resize; }
```

The default `cursor: auto` resolves to `text` over text and `default` (arrow) over everything else. Every interactive element that doesn't explicitly declare a cursor is lying to the user about whether it can be clicked.

## Icon Weight Reduction

Icons and text are rendered differently at equal opacity. Text is composed of thin strokes with generous negative space. Icons — even outline-style ones — often have thicker strokes and more filled area. At `opacity: 1`, an icon next to a label reads as bolder and heavier than intended, creating visual imbalance in the component.

Reduce icon opacity to 80–90% when paired with text. This is usually enough to bring the optical weight into balance.

```css
.nav-item .icon   { opacity: 0.85; }
.button .icon     { opacity: 0.80; }
.list-item .icon  { opacity: 0.85; }
```

Alternatively, use a slightly lighter weight icon variant if the icon library provides one (e.g., switching from `solid` to `regular` in Font Awesome, or from stroke-width `2` to `1.5` in Lucide).

### When not to reduce

Standalone icon buttons (no label) should stay at full opacity — the icon carries full communicative weight and needs full contrast. Only reduce when an icon is secondary to an adjacent text label.
