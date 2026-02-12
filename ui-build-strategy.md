# Open-Claw UI Build Strategy

## Context

This document evaluates the best forward-looking approach to building the adaptive learning UI defined in `adaptive-learning-system.md`, considering the current AI tooling landscape (2026) and lessons from production platforms like Synthesis.com.

---

## Complexity Assessment (Without AI Tools)

The spec defines three layers of UI complexity:

### Layer 1: Representation Engine (7 Content Renderers)

| Knowledge Type | What the UI Renders | Complexity |
|---|---|---|
| Structure/hierarchy | Self-building animated diagrams | Medium |
| Process/sequence | Step-by-step flow traces | Medium |
| Comparison/contrast | Side-by-side with diff highlighting | Low |
| Cause and effect | Animated arrows showing "if X then Y" | Medium |
| Definition/terminology | Text + concrete example card | Low |
| Pattern/principle | Multiple examples collapsing into one | Medium-High |
| Spatial/physical | Annotated interactive diagrams | High |

### Layer 2: Radial Knowledge Map

- Force-directed or radial graph layout
- 7 color states per node + glow/pulse/crack effects
- Progressive revelation with expansion animations
- Connection spark animations traveling along edges
- Zoom/pan/click interaction
- Real-time state updates as the learner progresses

### Layer 3: Interaction System (Detour/Interrupt/Resume)

- Lesson playback that can freeze at any point
- Overlay system with question zone sliding up
- Detour panel with mini-lessons inside itself
- Detour stack (up to 3 levels deep) — nested modal dialogs, each with own lesson state
- Rewind animations when returning from detours
- Celebration particle effects on verified understanding

### Raw Effort Estimate

| Component | Solo Dev | Small Team (2-3) |
|---|---|---|
| 7 content renderers | 2-3 weeks | 1-1.5 weeks |
| Radial knowledge map | 3-4 weeks | 2 weeks |
| Lesson view + interrupt/detour system | 2-3 weeks | 1-1.5 weeks |
| Progress tracking + celebrations | 1 week | 3-4 days |
| Integration | 2 weeks | 1 week |
| **Total** | **10-13 weeks** | **5-7 weeks** |

---

## Lessons from Synthesis.com

Synthesis (synthesis.com) is a production adaptive math tutor born from SpaceX's Ad Astra school. Key takeaways from studying their approach:

**Confirmed stack:** React + TypeScript, web + iPad, Canvas-based manipulatives

**Design decisions that simplify everything:**

1. **No knowledge graph visualization** — The AI tracks learner state behind the scenes. The student just sees the next lesson. This eliminates the hardest UI component.
2. **Conversational, not visual** — Primary interaction is text/voice back-and-forth. Manipulatives (grouping icons, number lines) are simple Canvas visuals, not complex animated diagrams.
3. **Scripted AI, not generative** — Pre-authored responses with branching logic to select the right one. Much simpler than real-time content generation.
4. **Gamification over visualization** — Instead of knowledge maps and animated renderers, the interaction itself is the game.

---

## Three Tiers of Approach

### Tier 1: "Ship in Days" — AI App Builders Generate the UI

Use AI-powered app builders to generate the majority of the UI from prompts.

| Tool | What It Does | Best For |
|---|---|---|
| [v0 by Vercel](https://v0.dev) | Prompt to production React + Tailwind components | Generating each of the 7 renderers as individual components |
| [Lovable](https://lovable.dev) | Prompt to full-stack React/TypeScript app with Supabase | Wiring up lesson view + progress tracking fast |
| [Bolt.new](https://bolt.new) | Prompt to running app in-browser | Quick throwaway prototypes to test interaction ideas |

**Workflow:**
1. Prompt Lovable: "Build a lesson player with a conversational AI tutor on the left, a canvas area for visual manipulatives on the right, and a progress bar at top"
2. Prompt v0: "A radial knowledge map component with animated nodes in 7 color states using React Flow"
3. Wire the generated code together in Cursor or Claude Code

**Reality check:** These tools generate ~80% of a UI fast, but the last 20% (adaptive logic, state machine, animation choreography) is still hand-written.

---

### Tier 2: "Ship in Weeks" — Generative UI (Recommended)

The agent dynamically decides what UI to show based on learner state at runtime.

**Core framework: [CopilotKit](https://www.copilotkit.ai)**

```
Learner answers wrong
  -> Agent decides: "they need a visual manipulative"
  -> Agent calls tool: renderManipulative({ type: "grouping", numbers: [3,7] })
  -> CopilotKit renders a pre-built React component with that data
  -> Learner interacts with it
  -> Agent observes the interaction and decides next step
```

This maps directly to the adaptive model in the spec. Instead of building a complex state machine with detour stacks, **the agent IS the state machine**.

#### Generative UI Patterns

| Pattern | How It Works | Fit for Open-Claw |
|---|---|---|
| **Static GenUI** | Pre-build 7 renderer components. Agent picks which to show and fills with data. | Best fit — controlled + adaptive |
| **Declarative GenUI** | Agent returns JSON describing UI layout. Frontend renders it. Uses Google A2UI or Open-JSON-UI spec. | Good for varying lesson layouts |
| **Open-ended GenUI** | Agent generates arbitrary UI. Maximum flexibility, minimum control. | Too risky for a learning app |

#### Recommended Stack

```
Next.js + React + TypeScript       (app framework)
CopilotKit                         (agent <-> UI runtime)
AG-UI Protocol                     (bidirectional agent-app communication)
Vercel AI SDK                      (LLM orchestration + tool calls + streaming)
7 pre-built renderer components    (one per knowledge type)
React Flow                         (knowledge map visualization)
Framer Motion                      (animations + transitions)
```

#### Why This Approach Wins

1. **Eliminates the hardest part of the spec.** The detour/interrupt/resume state machine disappears because the agent handles all branching logic. No XState needed.
2. **Renderers become tools.** Each knowledge type is a CopilotKit tool with a pre-built React component. The agent calls `renderProcessDiagram()` or `renderComparisonView()` as needed. Build 7 components, not a routing engine.
3. **Knowledge map becomes optional.** The agent tracks learner state internally. Add the visual map later as a "view into agent state" using `useCoAgentStateRender`, rather than building it as core architecture.
4. **Industry-aligned.** AG-UI protocol is adopted by Google, LangChain, AWS, Microsoft. Building on the winning standard.

---

### Tier 3: "Ship in a Day" — Follow Synthesis's Model (Simplest)

Skip visualization entirely. Build:

1. A **chat interface** with the AI tutor (conversational)
2. A **canvas area** for simple manipulatives (drag-and-drop, grouping)
3. A **progress tracker** (simple bar, not a knowledge graph)

Use Lovable or v0 to generate this in a single prompt session.

---

## Effort Comparison

| Approach | Solo Dev | Small Team (2-3) |
|---|---|---|
| Raw implementation (no AI tools) | 10-13 weeks | 5-7 weeks |
| With traditional libraries (React Flow, XState, etc.) | 4-5 weeks | 2-3 weeks |
| **Tier 2: CopilotKit + Generative UI (recommended)** | **3-4 weeks** | **1.5-2 weeks** |
| Tier 3: Synthesis-style minimal | 1 week | 2-3 days |

---

## Recommended Implementation Order (Tier 2)

| Phase | What to Build | Effort |
|---|---|---|
| 1 | Next.js + CopilotKit scaffold, agent backend with tool definitions | 2-3 days |
| 2 | Definition/terminology renderer (simplest, proves the routing works) | 1 day |
| 3 | Process/sequence renderer + comparison/contrast renderer | 2 days |
| 4 | Remaining 4 renderers (use v0 to generate initial versions) | 3-4 days |
| 5 | Lesson view with agent-driven flow (conversational + tool rendering) | 3-4 days |
| 6 | Knowledge map (React Flow + agent state via `useCoAgentStateRender`) | 1 week |
| 7 | Polish — animations (Framer Motion), celebrations, accessibility | 1 week |

---

## Key External Dependencies

| Library | Purpose | Link |
|---|---|---|
| CopilotKit | Agent-UI runtime, generative UI, shared state | [copilotkit.ai](https://www.copilotkit.ai) |
| AG-UI Protocol | Bidirectional agent-app communication standard | [copilotkit.ai/ag-ui](https://www.copilotkit.ai/ag-ui) |
| Vercel AI SDK | LLM orchestration, tool calls, streaming | [ai-sdk.dev](https://ai-sdk.dev) |
| React Flow | Knowledge map graph visualization | [reactflow.dev](https://reactflow.dev) |
| Framer Motion | Animations and transitions | [motion.dev](https://motion.dev) |
| canvas-confetti | Celebration particle effects | [github.com/catdad/canvas-confetti](https://github.com/catdad/canvas-confetti) |

---

## References

- [CopilotKit - Generative UI Framework](https://www.copilotkit.ai/generative-ui)
- [CopilotKit Developer's Guide to Generative UI in 2026](https://www.copilotkit.ai/blog/the-developer-s-guide-to-generative-ui-in-2026)
- [Google A2UI - Agent-Driven Interfaces](https://developers.googleblog.com/introducing-a2ui-an-open-project-for-agent-driven-interfaces/)
- [Best AI App Builder 2026 Comparison](https://getmocha.com/blog/best-ai-app-builder-2026/)
- [v0 vs Bolt vs Lovable Comparison](https://www.nxcode.io/resources/news/v0-vs-bolt-vs-lovable-ai-app-builder-comparison-2025)
- [Synthesis Tutor Review - Unite.AI](https://www.unite.ai/synthesis-tutor-review/)
- [Synthesis Tutor Review - Brighterly](https://brighterly.com/blog/synthesis-tutor-review/)
