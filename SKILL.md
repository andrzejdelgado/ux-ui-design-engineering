---
name: ux-ui-design-engineering
description: This skill guides creation of unique, production-grade frontend UIs that avoid generic "AI slop" looks, and help introduce design engineering principles for building interfaces that feel excellent. It helps to apply 33 principles usually omited in design/coding process.

User provides frontend requirements: a component, page, app, or UI to build or audit. They may include context about purpose, audience, or technical constraints.
---

# Design Thinking

When designing UI, optimize for precise, compounding details that make the product feel intentional from start to finish. Act like a master Senior Staff UX/UI Design Engineer.

A strong UI must:

- Anticipate user needs — functional, emotional, and aesthetic.
- Guide users with clarity so they immediately understand what to do next.
- Minimize effort, friction, hesitation, and unnecessary decision-making.
- Feel polished and deliberate across layout, spacing, typography, color, motion, hierarchy, and interaction states.
- Ensure every visual choice has a functional or communicative purpose.

**CRITICAL:**

- Choose a clear conceptual direction and execute with precision.
- Do not separate aesthetics from usability. Use visual design to communicate structure, priority, affordance, status, and trust.

Before finalizing any UI, ask:

1. Is the user's primary goal immediately clear?
2. Is the next action obvious without explanation?
3. Has unnecessary complexity been removed?
4. Do visual details reinforce meaning rather than decorate randomly?
5. Does the interface feel consistent, polished, and intentional?

## Framework & Design System Priority

Before applying any principle, check whether the project already defines rules in that area — Tailwind, shadcn/ui, Material UI, Radix, Primer, Chakra, or any in-house system. **Existing system conventions take priority.** Override only when deviation is intentional and documented. Inconsistency compounds; one off-spec component is worse than a slightly suboptimal default.

## Operating Protocol

This protocol defines how the agent should decide, edit, and verify.

1. Don't assume. Don't hide confusion. Surface tradeoffs.
2. Minimum code that solves the problem. Nothing speculative.
3. Touch only what you must. Clean up only your own mess.
4. Define success criteria. Loop until verified.

### Default work loop

**Inspect → Declare → Patch → Verify → Report.**

### UI success-criteria template

- The requested user-visible UI issue is resolved.
- The change follows existing framework, library, and design-system conventions.
- The change is scoped to the affected UI; no speculative abstractions.
- Accessibility preserved: semantics, keyboard, focus, hit areas, contrast, reduced-motion.
- Responsive behavior holds at supported breakpoints.
- Animation and rendering changes avoid known performance traps.

### Common operating mistakes

| Mistake | Fix |
| --- | --- |
| Agent invents missing design constraints | State uncertainty, inspect existing patterns, choose smallest reversible path |
| Broad refactor during a polish task | Touch only the affected component/styles |
| New abstraction added for a one-off UI fix | Prefer the minimum local change that satisfies the success criteria |
| Change shipped without verification | Define success criteria first, then loop only until those criteria pass |

## How to use this skill

This file is a router for visual principles, not a manual. For any task:

1. Apply the **Operating Protocol** above (always).
2. Use the **Routing** table below to identify which reference file(s) match the task. Open **only** those.
3. Skim the **Common Visual Mistakes** table — the highest-leverage list to keep in mind.
4. When delivering a review, follow **Review Output Format**.

## Routing

| Reference | Open when the task involves… |
| --- | --- |
| [typography.md](typography.md) | Font rendering / smoothing, type scale (`clamp`), line-height & letter-spacing, line length (`ch`), baseline alignment, OpenType features (`font-variant`, `font-feature-settings`, `tnum`, slashed zero, `ss0x`), tabular numbers, optical sizing (`opsz`), text decoration / underlines, cap-height trim, hanging punctuation. |
| [color.md](color.md) | Saturated neutrals, OKLCH tokens, P3 gamut, gradient techniques (color space, midpoint hints, banding fixes, mesh / radial stacks, hard stops, gradient text, conic, animated shimmer), `color-mix()`, container/surface brightness limits. |
| [surfaces.md](surfaces.md) | Concentric border radius (`outer = inner + padding`), optical vs geometric alignment, ≥40×40 hit areas, two-part elevation shadows (blur = 2× offset), inset/pressed states, `filter` color override, z-index scale, stacking contexts, `isolation`. |
| [visual-details.md](visual-details.md) | Visual hierarchy / de-emphasis, cursor affordance, icon optical weight (opacity 80–90%). |
| [animations.md](animations.md) | Initial-render animation skip, duration standards (150–400ms), easing direction (ease-out for enters), scroll-driven (`animation-timeline: view()`), `prefers-reduced-motion`, FLIP for layout, `linear()` springs, Framer Motion equivalents. |
| [performance.md](performance.md) | `transition: all` audit, `content-visibility: auto`, `contain`, compositor-only animations (`transform` + `opacity`), `will-change` discipline. |

When a topic spans files (e.g. animated gradients touch color, animations, and performance), open the file whose category is the *primary* concern, then cross-read only if needed.

## Common Visual Mistakes

| Mistake | Fix |
| --- | --- |
| Same border radius on parent and child | `outerRadius = innerRadius + padding` |
| Text looks heavy on dark backgrounds | `-webkit-font-smoothing: antialiased` on root |
| Icons read bolder than adjacent text | Reduce icon `opacity` to `0.85` |
| Pure grays feel cold next to brand colors | Tilt neutrals < 5% chroma toward the brand hue |
| Shadows look flat or glowy | Blur = 2× offset; stack one sharp directional + one soft diffuse layer |
| Animations play on initial page load | `initial={false}` (Framer) or gate animation class on first interaction |
| Layout animation janks | FLIP — animate `transform`, not `width`/`height`/`top`/`left` |
| `transition: all` causes hidden cost | Specify exact properties only |
| Z-index conflicts across components | Named token scale; never hardcode |
| Overrode an existing system convention | Check framework conventions before applying any principle |

## Review Output Format

Start with a short operating summary:

- **Success criteria** — what the UI must satisfy.
- **Assumptions / uncertainties** — anything inferred from the code or prompt.
- **Tradeoffs** — meaningful choices, especially consistency vs polish, performance vs richness, minimal scope vs refactor.
- **Verification** — checks run, static review performed, or limitations that prevented verification.

Then present every change as a markdown table grouped by topic, with **Before** and **After** columns. One row per discrete diff. Use the topic name as the heading (e.g. "Elevation Shadow", "Concentric Border Radius") — do **not** number principles. Omit topics where no change was needed.

### Example

#### Font Rendering
| Before | After |
| --- | --- |
| No smoothing on root element | Added `-webkit-font-smoothing: antialiased; -moz-osx-font-smoothing: grayscale; text-rendering: optimizeLegibility` to `:root` |

#### Concentric Border Radius
| Before | After |
| --- | --- |
| `rounded-xl` on card, `rounded-xl` on inner button with `p-2` | `rounded-2xl` on card (`12px + 8px = 20px`), `rounded-lg` on inner button |

#### Compositor-Only Animations
| Before | After |
| --- | --- |
| `transition: width 200ms` on expanding panel | Switched to FLIP — animate `transform: scaleX()` instead |
