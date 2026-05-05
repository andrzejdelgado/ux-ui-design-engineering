# UX UI Design Engineering

[![skills.sh](https://skills.sh/b/andrzejdelgado/ux-ui-design-engineering)](https://skills.sh/andrzejdelgado/ux-ui-design-engineering)

This skill closes that gap by packaging **33 design engineering principles** into an instruction set an AI coding assistant (Claude Code, Cursor, and others) loads when you ask it to do UI work — so the output is worth work of a seasoned UX/UI Design Engineer.

## Install

```bash
npx skills add andrzejdelgado/ux-ui-design-engineering
```

The skill installs into any agent — Claude Code, Cursor, GitHub Copilot, OpenCode, Goose, Amp, Codex, Gemini CLI, etc. After install, the agent loads the skill on demand whenever a task touches UI work.

## Three modes operation

- **Building from scratch** — uses design thinking principles, designs and codes UIs with intentional UX while appling all 33 principles.
- **Auditing existing work** — walks code or a design against the same checklist; returns a Before/After diff per topic.
- **Routing to save tokens and time** — split into a thin instruction file plus six topic references; the agent reads only what matches the task.

## Opearting Protocol

Underneath these modes sits an operating manifesto in the spirit of Andrej Karpathy's discipline for AI-assisted work:

* Don't assume, don't hide confusion, surface tradeoffs.
* Minimum code that solves the problem.
* Touch only what you must.
* Define success criteria, loop until verified, then stop.

## How it triggers

Once installed, the skill activates automatically when you ask the agent to:

- Build a new UI component, page, or app from scratch
- Polish an existing component ("make this feel better", "feels off")
- Audit frontend code or a design for visual quality
- Implement typography, color tokens, animations, shadows, or border radii
- Review a pull request that touches UI

You can also invoke it explicitly: ask the agent to "use the ux-ui-design-engineering skill" on a specific file or component.

## What it covers

33 principles across six areas. Each one targets a specific gap between a UI that works and one that feels finished. None require a design-system overhaul; most are a line or two of CSS.

| Category | Principles |
| --- | --- |
| **Typography** | Font rendering · type system · line-length constraint · baseline alignment · fluid type scale · OpenType features · text-decoration refinement · optical sizing |
| **Color** | Saturated neutrals · OKLCH for perceptual color · gradient refinement (color space, stop hints, banding fixes, mesh backdrops, conic UI) · container brightness limits |
| **Surfaces & depth** | Concentric border radius · optical alignment · minimum hit area · two-part elevation shadows · CSS filter color override · inset pressed states · named z-index scale |
| **Visual details** | De-emphasize to emphasize · cursor affordance · icon weight reduction |
| **Animations** | Skip-on-load · duration standards · ease-out for interactive · scroll-driven · reduced motion · FLIP for layout · spring physics via `linear()` |
| **Performance** | No `transition: all` · `content-visibility: auto` · CSS containment · compositor-only animations |

Each principle ships with code examples and common-mistake tables. See [SKILL.md](SKILL.md) for the operating manifesto and routing into the topic-specific reference files.

## Framework & design system priority

Consistency within an existing system matters more than applying any individual principle. When a project already uses Tailwind, shadcn/ui, Material UI, Radix, Chakra, or any other UI library, design system, that defines rules in a given area, those rules take priority, unless user decides differently.

## Repository layout

```
ux-ui-design-engineering/
├── SKILL.md              # Operating manifesto, routing table, common mistakes
├── typography.md         # Reference: typography deep-dive
├── color.md              # Reference: color, gradients, surface contrast
├── surfaces.md           # Reference: radius, alignment, shadows, z-index
├── visual-details.md     # Reference: hierarchy, cursor, icon weight
├── animations.md         # Reference: timing, easing, FLIP, springs
└── performance.md        # Reference: transition specificity, containment
```

The agent loads `SKILL.md` on activation and pulls in the relevant `*.md` reference file when a deeper dive is needed.

## License

MIT
