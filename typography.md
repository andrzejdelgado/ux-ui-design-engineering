# Typography

Font rendering, type scale, spacing, and decorative typographic details.

## Font Rendering

Browsers on macOS default to `subpixel-antialiased` — RGB channel hinting that makes text appear heavier and bolder than intended, especially on dark backgrounds. Override at the root level.

```css
:root {
  -webkit-font-smoothing: antialiased;    /* macOS Chrome, Safari */
  -moz-osx-font-smoothing: grayscale;    /* macOS Firefox */
  text-rendering: optimizeLegibility;    /* enables kerning + ligatures */
}
```

`text-rendering: optimizeLegibility` activates kerning pairs and common ligatures for supported typefaces. Note: on very long text blocks it can cause a minor performance cost — scope it to headings if needed.

ClearType on Windows operates at the OS level. These properties have no effect on Windows browsers.

## Tabular Numbers

Use `font-variant-numeric: tabular-nums` on any element where digits must align vertically or change in place — counters, prices, timers, dates and timestamps, data tables, charts and metric tiles, code/log line numbers, diff views, numeric form inputs. Without it, proportional figures shift surrounding characters as digits change width, which reads as instability.

Pair with a slashed-zero feature so `0` cannot be mistaken for `O`. The OpenType-standard tag is `"zero"` (`font-variant-numeric: slashed-zero`), but font support varies — Inter, for example, ships its slashed zero in stylistic set `"ss02"`, not `"zero"`. Verify rendered output.

```css
.counter,
.price,
.timer,
.data-cell { font-variant-numeric: tabular-nums slashed-zero; }

/* Inter: slashed zero lives in ss02 — needs font-feature-settings */
.code-display { font-feature-settings: "ss02"; }
```

Skip tabular figures for body prose with the occasional number — proportional figures read more naturally there. If the design already uses a monospaced typeface (Geist Mono, JetBrains Mono, Fira Code), tabular spacing is inherent — no override needed.

## Typography System

Font size alone does not create hierarchy. `line-height`, `letter-spacing`, and `font-weight` must each be set intentionally per context.

| Context        | line-height | letter-spacing | font-weight |
| -------------- | ----------- | -------------- | ----------- |
| Display heading | 1.1        | -0.02em        | 700         |
| Section heading | 1.25       | -0.01em        | 600         |
| Body           | 1.6         | 0              | 400         |
| Label / caption | 1.4        | 0.01em         | 500         |
| Monospace / code | 1.7       | 0              | 400         |

Tighter letter-spacing on large text prevents it reading as spaced-out. Slightly open spacing on small labels improves legibility. These are starting points — adjust optically per typeface.

## Line Length Constraint

Optimal reading comfort sits around 60–75 characters per line. Use `max-width: 70ch` on prose containers. `ch` is relative to the `0` glyph width of the current font — it scales correctly with font size changes.

```css
.prose { max-width: 70ch; }
```

Never let a body text column span the full viewport width. Wide columns force the eye to travel too far between lines and increase reading fatigue.

## Baseline Alignment

When mixing text of different sizes in a single row (e.g. a large price next to a currency symbol, or a heading next to a badge), align to the shared baseline — not the vertical center. Geometric centering creates a visible float effect that looks accidental.

```css
.price-row {
  display: flex;
  align-items: baseline;
}
```

## Fluid Type Scale

Scale type fluidly between breakpoints using `clamp()`. Always include a `rem`-based component alongside the viewport unit — `vw` alone breaks when users increase their browser's default font size.

```css
:root {
  --text-xs:  clamp(0.75rem,  1vw + 0.25rem, 0.875rem);
  --text-sm:  clamp(0.875rem, 1.5vw + 0.25rem, 1rem);
  --text-md:  clamp(1rem,     2vw + 0.5rem,  1.25rem);
  --text-lg:  clamp(1.25rem,  3vw + 0.5rem,  1.75rem);
  --text-xl:  clamp(1.5rem,   4vw + 0.5rem,  2.5rem);
  --text-2xl: clamp(2rem,     6vw + 0.5rem,  4rem);
}
```

Use a fluid type generator (Utopia, type-scale.com) to produce mathematically consistent scales.

## Cap-Height Trim

Buttons, badges, tags, and labels often look like their text is sitting slightly low inside the container. This is caused by half-leading — the browser adds space above the cap-height and below the baseline by default.

`text-box-trim` removes this space, enabling true optical centering.

```css
.badge,
.button-label,
.tag {
  text-box-trim: trim-both;
  text-box-edge: cap alphabetic;
}
```

Chrome 133+, Safari 18.2+. Treat as progressive enhancement — the fallback (standard half-leading) is acceptable, the enhancement is noticeably better.

## Font Feature Settings

Most typefaces ship with OpenType features disabled. Enable them explicitly where appropriate — but **prefer the high-level `font-variant` shorthand and its longhands** (`font-variant-caps`, `font-variant-numeric`, `font-variant-ligatures`, `font-variant-east-asian`) over `font-feature-settings`. The `font-variant` family composes correctly across cascading rules and produces more predictable results across fonts.

Reach for `font-feature-settings` only when no `font-variant` longhand exists for the feature you need — custom stylistic sets (`ss01`–`ss20`), character variants (`cv01`–`cv99`), or font-specific disambiguation features.

```css
/* Preferred: font-variant longhands */
.body { font-variant-ligatures: common-ligatures contextual; }
.ui-number { font-variant-numeric: lining-nums; }
.prose-number { font-variant-numeric: oldstyle-nums; }
.fraction { font-variant-numeric: diagonal-fractions; }

/* Fall back to font-feature-settings only for stylistic / character sets */
.code-display { font-feature-settings: "ss02"; }  /* Inter: slashed zero + disambiguation */
.alt-a        { font-feature-settings: "cv11"; }  /* Inter: single-storey 'a' */
```

Each OpenType tag is a four-character ASCII string, optionally followed by a positive integer or the keywords `on` / `off`. Font support varies — some fonts expose features under non-standard tags (e.g. Inter ships its slashed zero in `ss02`, not the standard `zero`). Always verify rendered output, not the spec.

Common features worth knowing:

| Tag | What it does |
| --- | --- |
| `liga` | Common ligatures (fi, fl, ff) — usually on by default |
| `calt` | Contextual alternates — on for body |
| `tnum` | Tabular figures — equal-width digits |
| `lnum` / `onum` | Lining vs oldstyle figures |
| `zero` | Slashed zero (where standard) |
| `frac` | Diagonal fractions |
| `ss01`–`ss20` | Font-specific stylistic sets |
| `cv01`–`cv99` | Character variants |

## Text Decoration Refinement

Browser default underlines sit too close to the text baseline and appear too thick. Control both properties explicitly on all links.

```css
a {
  text-decoration: underline;
  text-decoration-thickness: 1px;
  text-underline-offset: 0.2em;
}

a:hover {
  text-decoration-thickness: 2px; /* heavier on hover for feedback */
}
```

`text-underline-offset` is relative to the font — it scales correctly at all sizes.

## Hanging Punctuation

Opening quotation marks and closing punctuation optionally hang outside the text block margin, creating cleaner optical alignment for pull-quotes and blockquotes.

```css
blockquote,
.pull-quote {
  hanging-punctuation: first allow-end last;
}
```

Currently Safari-only. Treat as progressive enhancement — the fallback (flush punctuation) is standard, the enhancement produces typeset-quality output.

## Optical Sizing

Variable fonts that include an optical size axis (`opsz`) automatically adjust stroke contrast and detail based on rendered size. At small sizes: thicker strokes, reduced contrast, simplified details. At large sizes: higher contrast, finer details.

```css
:root { font-optical-sizing: auto; }
```

This is enabled by default in most browsers when a variable font is used, but setting it explicitly ensures consistency and overrides any inherited `none` value.
