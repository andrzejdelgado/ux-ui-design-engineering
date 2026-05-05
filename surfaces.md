# Surfaces & Depth

Border radius, optical alignment, hit areas, shadows, and z-index.

## Concentric Border Radius

When nesting rounded elements, the outer radius must be calculated from the inner radius plus the padding between them.

```
outerRadius = innerRadius + padding
```

### Examples

```css
/* Card containing a button */
.card   { border-radius: 20px; padding: 8px; }
.button { border-radius: 12px; } /* 20 - 8 = 12 */

/* Input inside a container */
.field-group { border-radius: 16px; padding: 6px; }
.input        { border-radius: 10px; } /* 16 - 6 = 10 */
```

When the padding exceeds 24px, treat the layers as independent surfaces — concentric math produces radii so small they become meaningless.

Mismatched radii on nested elements is the single most common thing that makes interfaces feel off. The outer element looks soft while the inner looks sharp, creating the impression of two unrelated components placed inside each other.

## Optical Over Geometric Alignment

Mathematical centering frequently produces visual misalignment. The geometric center of a bounding box is not the same as the optical center of the content inside it.

Common cases requiring optical correction:

- **Icon + text buttons** — icons have internal padding/whitespace in their SVG viewBox; they read as shifted even when geometrically centered
- **Play/arrow triangles** — the visual mass sits left of center; nudge 1–2px right
- **Asymmetric glyphs** — letters like `i`, `1`, or `!` appear off-center at large display sizes
- **Avatar + label rows** — a circular avatar reads as lower than text at the same `align-items: center`

Trust the eye over the coordinate. If it looks off by 1–2px, it is off.

## Minimum Hit Area

Every interactive element needs at least 40×40px of tappable/clickable area. This is a non-negotiable floor — not a target.

When the visible element is smaller (icon buttons, close icons, toggles), extend the hit area with a pseudo-element.

```css
.icon-button {
  position: relative;
  width: 24px;
  height: 24px;
}

.icon-button::before {
  content: "";
  position: absolute;
  inset: -8px; /* extends hit area to 40×40px */
}
```

Never let hit areas of two adjacent elements overlap — this creates accidental triggering and confusing affordances.

## Shadow Formula

The relationship between offset and blur determines whether a shadow reads as natural or artificial.

**Rule:** blur = 2× offset distance.

```css
/* Natural elevation */
box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12);   /* small */
box-shadow: 0 4px 8px rgba(0, 0, 0, 0.10);   /* medium */
box-shadow: 0 8px 16px rgba(0, 0, 0, 0.08);  /* large */
box-shadow: 0 16px 32px rgba(0, 0, 0, 0.06); /* floating */

/* Breaks the formula — reads as flat or glowy */
box-shadow: 0 4px 4px rgba(0, 0, 0, 0.12);   /* blur too low */
box-shadow: 0 4px 24px rgba(0, 0, 0, 0.12);  /* blur too high */
```

Reduce opacity as blur increases — larger, softer shadows should be more transparent to avoid making the UI feel heavy.

## Two-Part Shadow

A single shadow layer looks digital. Real objects cast two overlapping shadows: a sharp, directional one from the dominant light source, and a soft, diffuse one from ambient light.

```css
/* Small elevation — card */
.card {
  box-shadow:
    0 1px 2px rgba(0, 0, 0, 0.12),   /* directional: crisp, close */
    0 4px 12px rgba(0, 0, 0, 0.08);  /* diffuse: soft, far */
}

/* Large elevation — modal */
.modal {
  box-shadow:
    0 2px 4px rgba(0, 0, 0, 0.10),
    0 16px 48px rgba(0, 0, 0, 0.14);
}
```

In dark UIs, shadows are less effective because there is insufficient contrast between a dark shadow and a dark background. Use surface brightness elevation (slightly lighter background) instead of or alongside shadows.

## CSS Filter Color Override

When direct color manipulation is impossible — third-party components with hardcoded colors, embedded SVGs without accessible `fill`, icon fonts — use CSS `filter` to tint the element to a target color.

This is a precise technique. Use a filter generator tool (Barrett Sonntag's CSS filter generator is the reference implementation) to calculate the exact sequence from the element's current color to the target.

```css
/* Tint a black SVG icon to a brand blue */
.icon-brand {
  filter: invert(38%) sepia(93%) saturate(400%) hue-rotate(199deg) brightness(96%) contrast(101%);
}

/* When color is known but CSS variable is inaccessible */
.third-party-icon {
  filter: /* generated values for target color */;
}
```

Always document why `filter` is being used — it's a workaround, not a first-choice technique. Future maintainers need to understand the constraint.

## Inset Shadow for Pressed States

An inset shadow communicates physical depression — the element appears to push into the surface when activated. Pair with `scale-on-press` for full tactile feedback.

```css
.button:active {
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.12),
              inset 0 1px 2px rgba(0, 0, 0, 0.08);
  transform: scale(0.98);
}
```

The inset shadow alone works well on flat-design buttons where scale animation would be too strong. Use both together on primary CTAs where you want maximum tactile response.

## Elevation + Z-Index Scale

Never hardcode a z-index value. Define a named scale with stepped increments — gaps allow future layers to be inserted without rewriting the entire stack.

```css
:root {
  --z-base:     0;
  --z-raised:   100;   /* slightly elevated cards, sticky elements */
  --z-sticky:   500;   /* sticky headers, floating toolbars */
  --z-dropdown: 1000;  /* dropdowns, select menus, popovers */
  --z-overlay:  2000;  /* full-screen overlays, drawer backdrops */
  --z-modal:    3000;  /* modal dialogs */
  --z-toast:    4000;  /* toast notifications */
  --z-tooltip:  5000;  /* tooltips (must always be on top) */
}
```

### Stacking Context Awareness

New stacking contexts are created by: `opacity < 1`, `transform`, `filter`, `will-change`, `isolation: isolate`, `position: fixed/sticky`. An element inside a stacking context can never visually exceed its parent's z-index, regardless of its own z-index value.

Use `isolation: isolate` deliberately on component roots to contain their internal stacking without polluting the global z-index space.

```css
.dropdown-root { isolation: isolate; }
```
