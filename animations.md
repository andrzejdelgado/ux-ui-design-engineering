# Animations

Timing standards, easing, scroll animations, reduced motion, layout transitions, and spring physics.

## Skip Animation on Page Load

Users should not see content animating in when they first arrive on a page. Enter animations are for transitions triggered by user interaction — opening a menu, expanding an accordion, navigating between views. Running them on initial render makes the interface feel unstable and slow.

In Framer Motion, pass `initial={false}` to `AnimatePresence`:

```jsx
<AnimatePresence initial={false}>
  {isOpen && <Modal key="modal" />}
</AnimatePresence>
```

For CSS keyframe animations, use `animation-play-state` or conditionally add the animation class only after the first interaction.

## Animation Duration Standards

Duration should match the complexity and distance of the motion. Simple state changes need to feel instant. Complex enter sequences need enough time to be readable.

| Type | Duration |
| --- | --- |
| Micro interaction (hover, toggle, press) | 100–150ms |
| Simple state change (color, opacity, scale) | 150–200ms |
| Enter / exit transition (panel, dropdown) | 200–300ms |
| Complex enter sequence (staggered list) | 300–400ms total |
| Page-level transition | 300–500ms |
| Anything over 500ms | Reconsider — likely too slow |

These are guidelines for typical UI. Dense, data-heavy interfaces (dashboards, tables) should skew shorter. Expressive, marketing-oriented interfaces can skew slightly longer.

## Ease-out for Interactive

Easing communicates physics. `ease-out` (fast start, slow deceleration) mimics the behavior of an object responding to input — it moves immediately and settles gradually. This gives the impression of instant response.

```css
/* Interactive state changes */
.button   { transition: background-color 150ms cubic-bezier(0, 0, 0.3, 1); }
.dropdown { transition: transform 250ms cubic-bezier(0, 0, 0.3, 1); }

/* Exits — ease-in (slow start, fast end) */
.toast-exit { transition: opacity 200ms cubic-bezier(0.4, 0, 1, 1); }

/* Looping / ambient — ease-in-out */
.spinner { animation: spin 800ms cubic-bezier(0.4, 0, 0.6, 1) infinite; }
```

| Easing | Use for |
| --- | --- |
| `ease-out` | Enters, interactive responses, things arriving |
| `ease-in` | Exits, things leaving |
| `ease-in-out` | Looping, ambient, non-interactive motion |
| `linear` | Progress bars, timelines, scroll-linked |

## Scroll-Driven Animations

Animate elements as they enter the viewport using the native CSS scroll timeline API — no JavaScript, no IntersectionObserver, no library.

```css
@keyframes fade-up {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.card {
  animation: fade-up linear both;
  animation-timeline: view();
  animation-range: entry 0% cover 35%;
}
```

`animation-range: entry 0% cover 35%` means: start the animation when the element begins entering the viewport, finish it when 35% of the element is covered by the scroll container.

Use `@supports (animation-timeline: scroll())` to gate on supporting browsers. Chrome 115+, Firefox 110+. Safari support is in development.

### Reading progress indicator

```css
.progress-bar {
  transform-origin: left;
  animation: grow linear;
  animation-timeline: scroll(root);
}

@keyframes grow {
  from { transform: scaleX(0); }
  to   { transform: scaleX(1); }
}
```

## Reduced Motion

`prefers-reduced-motion: reduce` is set by users with vestibular disorders, motion sensitivity, or epilepsy. Vestibular disorders affect approximately 35% of adults over 40. This is not an edge case.

Provide reduced motion alternatives for every animation. Swap translate/scale motion for opacity-only fades — they convey transition without spatial movement.

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

For more granular control — preserving opacity transitions while removing motion:

```css
@media (prefers-reduced-motion: reduce) {
  .animated-card {
    /* Remove motion */
    transform: none !important;
    /* Keep fade */
    transition: opacity 200ms ease-out;
  }
}
```

In Framer Motion, use `useReducedMotion()`:

```jsx
const prefersReducedMotion = useReducedMotion()

const variants = {
  hidden: { opacity: 0, y: prefersReducedMotion ? 0 : 20 },
  visible: { opacity: 1, y: 0 }
}
```

## FLIP for Layout Animations

Animating layout properties (`width`, `height`, `top`, `left`, `margin`, `padding`) triggers full layout recalculation on every frame. At 60fps this causes jank. The FLIP technique moves layout animations to the compositor thread by animating `transform` instead.

**F**irst → record starting position with `getBoundingClientRect()`
**L**ast → apply the new state, record ending position
**I**nvert → apply a transform that puts the element back to its starting visual position
**P**lay → animate the transform to `none` (identity)

```js
// 1. First
const first = element.getBoundingClientRect()

// 2. Last — apply the layout change
element.classList.add('expanded')
const last = element.getBoundingClientRect()

// 3. Invert
const dx = first.left - last.left
const dy = first.top - last.top
element.style.transform = `translate(${dx}px, ${dy}px)`

// 4. Play
requestAnimationFrame(() => {
  element.style.transition = 'transform 300ms ease-out'
  element.style.transform = ''
})
```

Framer Motion's `layout` prop implements FLIP automatically:

```jsx
<motion.div layout>
  {/* content */}
</motion.div>
```

Use FLIP for: expanding/collapsing panels, reordering lists, moving cards between columns, shared element transitions.

## Spring Physics via `linear()`

CSS `linear()` generates custom easing functions that can simulate spring and bounce physics without a motion library.

```css
/* Natural spring — slight overshoot and settle */
.popover {
  transition: transform 500ms linear(
    0, 0.009, 0.035 2.1%, 0.141, 0.281 6.7%,
    0.723 12.9%, 0.938 16.7%, 1.017, 1.077,
    1.106, 1.121 24.2%, 1.121, 1.101,
    1.058 32.9%, 1.015 39%, 1 44%,
    0.994 51.5%, 0.998 61%, 1
  );
}
```

Use a spring generator tool (linear-easing-generator.netlify.app) to produce the `linear()` values from spring parameters (stiffness, damping, mass).

If the project has `motion` or `framer-motion` in `package.json`, use their spring API instead — identical result, cleaner syntax:

```jsx
<motion.div
  animate={{ scale: 1 }}
  transition={{ type: "spring", stiffness: 300, damping: 25, bounce: 0 }}
/>
```

`bounce` must always be `0` or close to `0` for UI transitions. Visible bounce is appropriate for game UI or expressive marketing — not for product interfaces.
