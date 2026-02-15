# Agent skills

Shared skill library for [Heirloom Logic LLC](https://heirloomlogic.com) projects. Skills extend AI coding agents with domain knowledge, style enforcement, and reference material.

Most of these are other people's work, pulled in as needed. Primary sources are [erichowens/some_claude_skills](https://github.com/erichowens/some_claude_skills/tree/main/.claude/skills) and [mcpmarket.com](https://mcpmarket.com/tools/skills). A few are ours (`dehumanizer`, `swidux-ref`). Project-specific skills stay in their own repos.

MIT licensed. Use what you want.

## What's a skill

A folder with a `SKILL.md` file. The frontmatter defines the name, description, and activation triggers. The markdown body contains the actual instructions, examples, and reference material the agent consults at runtime.

Some skills include scripts, examples, or resource files alongside the main doc. The agent reads what it needs when it needs it.

## Skills

### Apple platform

| Skill | What it does |
|---|---|
| `swidux-ref` | Architecture reference for Swidux (Redux-style state management for SwiftUI + SwiftData) |
| `swift6-concurrency` | Diagnose and fix Swift 6 strict concurrency errors, including Approachable Concurrency (SE-0470, SE-0461) |
| `swiftui-animation-ref` | Animation reference covering `Animatable`, `@Animatable` macro, springs, timing curves — iOS 13 through iOS 26 |
| `swiftui-debugging` | Diagnostic decision trees for view update bugs, preview crashes, and layout issues |
| `swiftui-gestures` | Gesture recognition, composition, state management, and conflict resolution |
| `swiftui-layout` | Adaptive layout patterns — `ViewThatFits`, `AnyLayout`, size classes, iOS 26 free-form windows |
| `swiftui-nav-ref` | Navigation API reference — `NavigationStack`, `NavigationSplitView`, deep linking, Liquid Glass (iOS 26) |

### Graphics and simulation

| Skill | What it does |
|---|---|
| `metal-shader-expert` | Metal shaders, TBDR architecture, PBR rendering, GPU profiling |
| `physics-rendering-expert` | Real-time rope/cable physics — PBD, Verlet integration, constraint solvers, tangling detection |

### Design

| Skill | What it does |
|---|---|
| `adhd-design-expert` | UX for ADHD brains — cognitive load reduction, time blindness, dopamine-driven engagement |
| `color-theory-palette-harmony-expert` | Perceptual color science for computational photo composition — palette harmony, warm/cool alternation, hue sequencing |
| `native-app-designer` | iOS/Mac and web apps with organic, non-AI aesthetic — SwiftUI, React animations, physics-based motion |

### Writing and docs

| Skill | What it does |
|---|---|
| `dehumanizer` | Strip AI writing patterns. Gen-X voice: deadpan, economical, direct |
| `markdown-doc-surgeon` | Tighten bloated documentation without losing information |
| `skill-writer` | Create new agent skills with correct structure and frontmatter |

### Code quality

| Skill | What it does |
|---|---|
| `refactoring-surgeon` | Refactor code without changing behavior — extract methods, eliminate smells, enforce SOLID |

### AI engineering

| Skill | What it does |
|---|---|
| `prompt-refinement` | Transform vague prompts into structured TCRO format with phase-specific clarification |

### Business

| Skill | What it does |
|---|---|
| `indie-monetization-strategist` | Monetization for indie devs and small teams — freemium, SaaS pricing, sponsorships, passive income |

## Adding a skill

Create a new folder, add a `SKILL.md` with frontmatter and instructions. Read the `skill-writer` skill if you want guidance on structure. Or don't — the format is straightforward enough to figure out from the existing examples.

## License

MIT. See [LICENSE](LICENSE).
