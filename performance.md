# Performance

Transition specificity, compositor-only animations, and CSS containment.

## Never Use `transition: all`

`transition: all` registers a watcher on every animatable CSS property. Any property change — including ones you didn't intend to animate — triggers a transition. This creates unexpected animations and unnecessary computation on every render cycle.

```css
/* Bad — watches ~50 animatable properties */
.button { transition: all 200ms ease-out; }

/* Good — watches only what you need */
.button {
  transition:
    background-color 150ms ease-out,
    transform 150ms ease-out,
    box-shadow 150ms ease-out;
}
```

`transition-transform` is shorthand for `transform, translate, scale, rotate` — use it when animating individual transform components independently (CSS individual transforms, not the shorthand `transform` property).

```css
/* Animating individual transform components */
.card {
  transition: scale 150ms ease-out, translate 200ms ease-out;
}
```

## `content-visibility: auto`

`content-visibility: auto` tells the browser to skip layout and paint for elements outside the viewport until they approach. The browser still reserves space for them, but defers the rendering work.

This is measurably effective on long pages — feed layouts, dashboards with many cards, long lists, articles with many sections.

```css
.card,
.list-item,
.section {
  content-visibility: auto;
  contain-intrinsic-size: auto 320px; /* estimated height — prevents scrollbar jump */
}
```

Always pair with `contain-intrinsic-size`. Without it, the browser renders the element at 0 height until it enters the viewport, causing the scrollbar to jump as the user scrolls.

`auto` in `contain-intrinsic-size: auto 320px` means: use the remembered size from the last render if available, fall back to `320px` otherwise. This minimizes scrollbar instability after the first scroll through.

Do not apply to elements near the top of the page or elements that are always visible — the containment overhead outweighs the benefit.

## CSS Containment

`contain` tells the browser that an element's internal layout, paint, and style are isolated from the rest of the document. This allows the browser to skip expensive cross-tree recalculations when the element changes.

```css
/* Full containment shorthand */
.widget { contain: content; }
/* Equivalent to: contain: layout paint style */

/* Layout only — element's size doesn't affect surrounding layout */
.fixed-size-card { contain: layout; }

/* Paint only — content clipped to border-box, no overflow rendering */
.clipped-panel { contain: paint; }
```

| Value | What it does |
| --- | --- |
| `layout` | Element's internals don't affect external layout |
| `paint` | Content clipped to border-box; compositing layer hint |
| `style` | CSS counters and quotes scoped to element |
| `size` | Element's size is independent of its content |
| `content` | Shorthand for `layout paint style` |
| `strict` | Shorthand for `layout paint style size` |

Use `contain: content` on self-contained components — cards, panels, widgets, list items — where internal changes should never affect surrounding layout.

## Compositor-Only Animations

The browser rendering pipeline has three phases:

1. **Layout** — calculate element sizes and positions
2. **Paint** — fill pixels for each element
3. **Composite** — layer and display the final result on screen

Only `transform` and `opacity` skip phases 1 and 2 entirely and run on the GPU compositor thread. Every other animated property forces a full or partial layout/paint recalculation on every frame.

| Property | Layout | Paint | Compositor |
| --- | --- | --- | --- |
| `transform` (translate, scale, rotate) | — | — | ✓ |
| `opacity` | — | — | ✓ |
| `background-color`, `color` | — | ✓ | — |
| `box-shadow`, `border`, `outline` | — | ✓ | — |
| `width`, `height`, `padding`, `margin` | ✓ | ✓ | — |
| `top`, `left`, `right`, `bottom` | ✓ | ✓ | — |
| `font-size`, `line-height` | ✓ | ✓ | — |

For motion effects, animate only `transform` and `opacity`. Never animate `width`, `height`, `top`, `left`, `margin`, `padding`, `box-shadow`, or `border` directly. Use the FLIP technique (see animations.md) to convert layout animations to transform-based ones.

```css
/* Bad — triggers layout on every frame */
.panel { transition: height 300ms ease-out; }

/* Good — compositor only */
.panel { transition: transform 300ms ease-out; }
/* Use scaleY() or translateY() to simulate height change */
```

### `will-change`

`will-change` promotes an element to its own compositor layer before an animation begins, eliminating the first-frame stutter on complex transitions.

```css
.modal { will-change: transform, opacity; }
```

Use sparingly:
- Only for `transform`, `opacity`, or `filter` — never `will-change: all`
- Only when you observe a first-frame stutter without it — not pre-emptively
- Remove after the animation completes if applied dynamically

```js
element.addEventListener('animationend', () => {
  element.style.willChange = 'auto'
})
```

Overusing `will-change` causes the browser to promote too many elements to separate GPU layers, consuming significant memory and potentially degrading performance rather than improving it.
