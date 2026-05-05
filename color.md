# Color

Saturated neutrals, perceptual color spaces, and surface contrast limits.

## Saturated Neutrals

Pure mathematical grays feel cold and disconnected from the surrounding palette. When you mix a warm brand color with a neutral gray sidebar, the gray reads as foreign — like it belongs to a different design.

Infuse grays with a slight hue tilt matching the dominant brand color. Keep saturation below 5% — enough to create cohesion, invisible as a deliberate choice.

```css
/* Pure gray — feels cold */
--gray-500: oklch(55% 0 0);

/* Saturated neutral — cohesive */
--gray-500: oklch(55% 0.012 250); /* slight blue tilt for a blue brand */
```

Commit to either warm or cool saturation — never mix both within the same neutral scale.

## OKLCH for Perceptual Color

`hsl()` and hex are not perceptually uniform. Changing lightness by 10 points in HSL produces different visual steps depending on the hue — yellows appear much lighter than blues at the same L value. This makes systematic palette manipulation unpredictable.

OKLCH (Oklab Chroma Lightness Hue) is perceptually uniform. Equal numerical steps produce equal visual steps across all hues. It also natively accesses the P3 display gamut available on modern screens.

```css
:root {
  /* Primary */
  --color-primary:       oklch(55% 0.18 264);
  --color-primary-hover: oklch(48% 0.18 264); /* only L changes */
  --color-primary-muted: oklch(55% 0.06 264); /* only C changes */

  /* Neutral scale — consistent L steps */
  --gray-100: oklch(96% 0.012 250);
  --gray-300: oklch(80% 0.012 250);
  --gray-500: oklch(60% 0.012 250);
  --gray-700: oklch(40% 0.012 250);
  --gray-900: oklch(18% 0.012 250);
}
```

Use OKLCH for all design tokens and any runtime color manipulation (lightening, darkening, opacity variants). Convert legacy hex/HSL values to OKLCH at the token layer.

### P3 Gamut

OKLCH automatically enables P3 colors when lightness and chroma push beyond the sRGB boundary. Modern displays (most MacBooks, iPhones, iPads) can render these — sRGB clips them to a duller equivalent.

```css
/* This color is outside sRGB — browsers on P3 displays render the richer version */
--color-accent: oklch(65% 0.28 145); /* vivid green */
```

No special fallback needed — browsers on sRGB displays render the closest in-gamut equivalent automatically.

## Gradient Refinement

Refined gradients are not single linear blends — they're a small toolkit of techniques. Color space defines the path between two stops, but six other techniques do the rest of the heavy lifting on real product surfaces.

### Color space

When interpolating between two colors (gradients, theme transitions, `color-mix()`), the color space defines the path. The default sRGB drags vivid → vivid transitions through a muddy mid-grey, killing saturation at the midpoint. Specify a perceptual color space explicitly: `oklab` stays in pastel territory and avoids the dead zone; `oklch longer hue` walks the hue wheel and produces the full rainbow between any two hues.

```css
.gradient-oklab { background: linear-gradient(in oklab to right, oklch(70% 0.30 330), oklch(85% 0.30 130)); }
.gradient-oklch { background: linear-gradient(in oklch longer hue to right, oklch(70% 0.30 330), oklch(85% 0.30 130)); }

/* color-mix() follows the same rule */
.tint { background: color-mix(in oklab, var(--brand) 70%, white); }
```

### Color-stop hint (midpoint shift)

A bare percentage between two color stops moves the perceptual midpoint of the transition without adding a third color. Use it to bias the blend toward one end while keeping a single smooth gradient — useful for backdrops where one color should dominate visual weight.

```css
/* Midpoint at 25% — gradient leans heavily into the red end */
.bias { background: linear-gradient(in oklch to right, oklch(60% 0.20 30), 25%, oklch(60% 0.20 264)); }
```

### Banding remediation

Subtle dark gradients (low contrast, large area) show stepping artifacts on 8-bit displays. Two fixes that compose well:

1. **Extra stops along the curve** — insert intermediate color stops at small variations so the eye sees a smoother curve. Best for synthetic/clean surfaces.
2. **Grain overlay** — overlay a low-opacity noise texture; the grain breaks the eye's pattern recognition that makes banding perceptible. Best for editorial or photo-adjacent UI.

```css
/* (1) Extra stops smooth low-contrast banding */
.bg-clean {
  background: linear-gradient(in oklch to bottom,
    oklch(20% 0.01 264) 0%,
    oklch(18% 0.01 264) 25%,
    oklch(15% 0.01 264) 60%,
    oklch(12% 0.01 264) 100%);
}

/* (2) Grain overlay — perceptual fix on photographic surfaces */
.bg-grain {
  background:
    url("/noise.png"),
    linear-gradient(in oklch to bottom, oklch(20% 0.01 264), oklch(12% 0.01 264));
  background-blend-mode: overlay;
  background-size: 128px, cover;
}
```

### Stacked radial gradients (mesh-like backdrops)

A single linear gradient feels flat. Stack 3–4 radial gradients at different positions, sizes, and low alpha values for a soft, ambient-light backdrop that reads as three-dimensional without explicit shadows. Scales from product hero sections to dashboard backgrounds.

```css
.hero {
  background:
    radial-gradient(60% 50% at 18% 22%, oklch(72% 0.18 264 / 0.35), transparent 70%),
    radial-gradient(50% 60% at 82% 28%, oklch(74% 0.16 320 / 0.30), transparent 70%),
    radial-gradient(70% 60% at 50% 90%, oklch(76% 0.20 180 / 0.28), transparent 70%),
    oklch(98% 0.005 264);
}
```

### Hard stops for clean splits

Two stops at the same position produce a sharp color edge with no transition — useful for divider lines, two-color backgrounds, or tab indicators painted into the background instead of as a separate element.

```css
.split { background: linear-gradient(to right, oklch(50% 0.18 264) 50%, oklch(60% 0.18 25) 50%); }
```

### Gradient text

For accent headings and brand wordmarks. Always pair `background-clip: text` with `-webkit-text-fill-color: transparent` for Safari, and use a perceptual color space for vivid → vivid transitions.

```css
.headline {
  background: linear-gradient(in oklch to right, oklch(60% 0.22 264), oklch(60% 0.22 320));
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
```

### Conic gradients for radial UI

Progress rings, color wheels, dial controls, and segmented donuts don't need SVG. `conic-gradient()` delivers the same effect natively, with hard stops for segments or smooth transitions for spinners.

```css
/* Progress ring — 75% complete */
.ring {
  background: conic-gradient(from 0deg, oklch(60% 0.20 264) 0% 75%, oklch(94% 0.004 264) 75% 100%);
}

/* Loading spinner — fades around the wheel; mask carves out the ring shape */
.spinner {
  background: conic-gradient(from 0deg, transparent, oklch(60% 0.20 264));
  -webkit-mask: radial-gradient(circle, transparent 60%, black 60%);
          mask: radial-gradient(circle, transparent 60%, black 60%);
}
```

### Animated gradients

Animate `background-position` (not `background-image`) on an oversized gradient for shimmer and breathing-glow effects. Stays compositor-only — see the Performance reference for why that matters.

```css
.shimmer {
  background: linear-gradient(in oklch 110deg, oklch(96% 0.01 264), oklch(99% 0.005 264), oklch(96% 0.01 264));
  background-size: 220% 100%;
  animation: shimmer 1.6s linear infinite;
}
@keyframes shimmer { from { background-position: 0% 0; } to { background-position: -220% 0; } }
```

## Container Brightness Limits

Surfaces in a UI create depth through brightness differentiation. But too much differentiation makes containers compete with content for visual attention. Too little makes them invisible.

**Dark UI:** maintain a maximum 12% brightness difference between a container and its background.
**Light UI:** maintain a maximum 7% brightness difference.

```css
/* Dark UI example */
--background:  oklch(12% 0.012 250); /* base */
--surface:     oklch(18% 0.012 250); /* +6% L — within the 12% limit */
--surface-raised: oklch(22% 0.012 250); /* +4% L above surface */

/* Light UI example */
--background:  oklch(98% 0.008 250);
--surface:     oklch(94% 0.008 250); /* -4% L — within the 7% limit */
```

Beyond these limits, containers draw the eye away from content. Below them, the visual separation becomes imperceptible — content appears to float on a flat plane.
