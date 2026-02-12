# Adaptive Learning System — Unified Reference

## Table of Contents

1. [Introduction and Design Philosophy](#1-introduction-and-design-philosophy)
2. [The Knowledge Map](#2-the-knowledge-map)
3. [The Learner Model](#3-the-learner-model)
4. [Content Ingestion — Teaching the System a New Domain](#4-content-ingestion--teaching-the-system-a-new-domain)
5. [The Diagnostic Engine](#5-the-diagnostic-engine)
6. [The Teaching Engine](#6-the-teaching-engine)
7. [Agent Architecture](#7-agent-architecture)
8. [The End-to-End Workflow](#8-the-end-to-end-workflow)
9. [Persistent Memory and State](#9-persistent-memory-and-state)
10. [Interaction Design](#10-interaction-design)
11. [Reference: What the System Tracks](#11-reference-what-the-system-tracks)
12. [System Self-Improvement — Feedback Loops](#12-system-self-improvement--feedback-loops)
13. [Error Recovery and Graceful Degradation](#13-error-recovery-and-graceful-degradation)

---

## 1. Introduction and Design Philosophy

### What This System Is

This is not a tutoring system that happens to diagnose — it is a **cognitive diagnostic system** that happens to teach. The distinction matters. A traditional tutor follows a curriculum and hopes the learner keeps up. This system models the learner's understanding as a living data structure, identifies precisely where knowledge breaks down, and repairs understanding from the foundation up.

The system is **domain-agnostic**. Whether teaching organic chemistry, software architecture, music theory, or automotive repair, the same principles apply: knowledge has structure, understanding has gaps, and those gaps can be located through strategic questioning rather than exhaustive testing.

### The Core Insight

When someone says "I don't understand photosynthesis," the problem could be at any level. Maybe they don't understand what chloroplasts are (a structural gap). Maybe they understand the inputs and outputs but can't connect them to cellular respiration (a synthesis gap). Maybe they think photosynthesis produces oxygen *from* carbon dioxide rather than from water (a misconception).

The system's job is to find **where** the gap is, not just **that** there is one. Diagnosis before teaching. Always.

### Design Principles

These principles govern every decision the system makes:

1. **Diagnose first, teach second.** Never explain something until you know exactly what the learner is missing. A well-placed question is worth more than a well-crafted explanation.

2. **Teach at the detail level.** Don't explain a high-level concept when the real problem is a missing foundational piece. Find the atomic knowledge unit that's broken and fix that.

3. **Repair inward.** Fix details and foundations, then test whether concepts are restored, then test whether cores are restored. Never skip levels.

4. **Prioritize by impact.** A gap that blocks understanding of three higher-level topics is more important than a gap that blocks one. Teach what unlocks the most.

5. **Misconceptions over gaps.** A gap is missing knowledge — the learner knows they don't know. A misconception is wrong knowledge the learner believes is right. Misconceptions are more dangerous because they silently corrupt understanding of everything built on top of them.

6. **Confidence decays.** Verified knowledge fades over time. A concept proven understood three months ago may need re-verification. This is spaced repetition built into the knowledge model.

7. **Match representation to knowledge type.** Structure should be shown as diagrams. Processes should be traced step by step. Comparisons should be shown side by side. The medium must fit the message.

8. **The learner sees their map.** The knowledge map is not a hidden internal data structure — the learner watches it grow, sees where they're strong, sees where the gaps are. Transparency turns learning into navigation.

---

## 2. The Knowledge Map

### Knowledge as a Radial Map

All knowledge in a domain is modeled as a **directed acyclic graph (DAG)** internally, but visualized as a **radial knowledge map** — a mind-map-like layout where broad topics sit at the center and specifics radiate outward. This radial layout separates two hierarchies that point in opposite directions:

- **Distance from center** = abstraction level (center = broad, periphery = specific)
- **Directed arrows** = dependency (what must be understood first)

The map has four types of nodes:

| Type | Name | Description | Example (Biology) | Example (Software) | Example (Music) |
|------|------|-------------|-------------------|--------------------|--------------------|
| **Core** | Major topic | A broad area of understanding, sitting at the center of the map | Cellular Respiration | How the API Gateway Routes Requests | Harmonic Progression |
| **Concept** | Supporting concept | A cluster of related ideas needed by a core. Concepts can nest — a concept can contain sub-concepts, enabling **variable depth**. | Glycolysis | WebSocket Protocol | Chord Function |
| **Detail** | Atomic knowledge unit | The smallest teachable piece of understanding | What glucose is | Persistent vs. stateless connections | Tonic resolution |
| **Foundation** | Cross-cutting prerequisite | A prerequisite that supports nodes across multiple cores. Visually distinct — drawn with connections spanning different regions of the map. | ATP as energy currency | HTTP request/response cycle | Interval recognition |

A core is understood when its concepts are understood. A concept is understood when its sub-concepts and details are understood. Teaching always targets details. Foundations are taught whenever any dependent node needs them — and once taught, all dependent nodes across the map benefit.

### The Hierarchy in Practice

```
KNOWLEDGE MAP (the whole radial graph)
└── CORE: How Cellular Respiration Works
    ├── CONCEPT: Glycolysis
    │   ├── DETAIL: What glucose is
    │   ├── DETAIL: Net energy yield
    │   └── CONCEPT: Enzyme catalysis              ← sub-concept (variable depth!)
    │       ├── DETAIL: Lock-and-key model
    │       └── DETAIL: Activation energy
    ├── CONCEPT: The Krebs Cycle
    │   ├── DETAIL: Acetyl-CoA
    │   ├── DETAIL: Electron carriers (NADH, FADH2)
    │   └── DETAIL: CO2 release
    ├── CONCEPT: Oxidative Phosphorylation
    │   ├── DETAIL: Electron transport chain
    │   ├── DETAIL: Chemiosmosis
    │   └── DETAIL: Oxygen as final electron acceptor
    └── CONCEPT: Why Cells Need Energy
        ├── DETAIL: Active transport
        ├── DETAIL: Protein synthesis
        └── DETAIL: Cell division

FOUNDATION: "ATP as energy currency"
  → supports Glycolysis (hard dependency)
  → supports Active Transport in a different Core (hard dependency)
```

Note how "Enzyme catalysis" is a **sub-concept** within Glycolysis, not a detail — it has its own details underneath it. This variable depth means the map can model knowledge at whatever granularity the domain requires, without being forced into exactly three levels.

The same pattern applies regardless of domain. A software architecture topic:

```
KNOWLEDGE MAP
└── CORE: How the API Gateway Routes Requests
    ├── CONCEPT: WebSocket Protocol
    │   ├── DETAIL: TCP connections
    │   ├── DETAIL: Persistent vs. stateless connections
    │   └── DETAIL: Bidirectional messaging
    ├── CONCEPT: Session Management
    │   ├── DETAIL: Session keys
    │   ├── DETAIL: State isolation
    │   └── DETAIL: History management
    ├── CONCEPT: Routing Logic
    │   ├── DETAIL: Method-based dispatch
    │   ├── DETAIL: Channel-to-agent mapping
    │   └── DETAIL: Response delivery
    └── CONCEPT: Security Model
        ├── CONCEPT: Network Isolation              ← sub-concept
        │   ├── DETAIL: Network interfaces
        │   └── DETAIL: Localhost binding
        └── DETAIL: Tunnel exposure

FOUNDATION: "HTTP request/response cycle"
  → supports Routing Logic (hard dependency)
  → supports REST API Design in a different Core (hard dependency)
```

### Dependency Weights

Not all connections between nodes are equal. Each dependency carries a weight:

- **Hard dependency** — Cannot understand the parent without this child. "You cannot understand oxidative phosphorylation without understanding what an electron carrier is." The system **must** teach the child before the parent.

- **Helpful dependency** — Makes the parent easier to understand, but you can get by with a simplified mental model. "Understanding enzyme kinetics helps with glycolysis, but you can learn glycolysis with just 'enzymes speed up reactions.'" The system may teach the parent with a simplified stand-in and revisit the detail later.

- **Optional dependency** — Nice to know, enriches understanding, but not blocking. "Knowing the evolutionary history of mitochondria is interesting but not required to understand cellular respiration." The system skips these unless the learner asks or has extra time.

This three-tier dependency model prevents the system from requiring learners to know everything before they can learn anything. It distinguishes between "you literally cannot proceed without this" and "this would be nice context."

### Cross-Topic Connections and Foundations

Because knowledge is a graph rather than a strict tree, the same node can serve multiple concepts and even multiple cores. This is where **foundations** become important.

A foundation is a cross-cutting prerequisite that supports nodes across different regions of the map. For example:

- "ATP as energy currency" is a foundation — it supports Glycolysis, Active Transport, and Muscle Contraction (different cores entirely). It doesn't belong to any single core; it underlies all of them.
- "HTTP request/response cycle" is a foundation — it supports WebSocket Protocol, REST API Design, and Authentication Flows across multiple cores.

Regular details can also be shared — "Persistent connections" is a detail under WebSocket Protocol, but it's also needed by Real-Time Streaming and Event-Driven Architecture.

When the system teaches a shared detail or foundation, it teaches it once but marks it as relevant across all its dependent contexts. When it's verified, all dependents benefit. When it decays, all dependents are affected. Foundations receive distinct visual treatment on the map — they appear as nodes with connections spanning multiple regions, making their cross-cutting importance visible.

### Progressive Revelation

The full knowledge map is never shown to the learner all at once. The map **expands outward** as the learner is assessed and taught:

1. **Initial state**: The learner sees only core nodes at the center — major topics, dimly connected.
2. **After assessment**: Cores the learner claimed familiarity with expand, revealing their concepts. Each concept is color-coded by the assessment result.
3. **During teaching**: As the system probes a concept, its details appear around it. Dependency connections draw themselves across the map, showing how foundations support multiple structures.
4. **Over time**: The map fills outward from center like expanding knowledge — dark areas illuminate, isolated clusters connect via shared foundations, and the learner can see the shape of their understanding.

This progressive revelation serves two purposes: it avoids overwhelming the learner with the full scope of what they don't know, and it creates a visceral sense of growth as the map fills outward. Critically, the **visual direction matches the teaching direction**: details are taught first (at the periphery) and understanding propagates inward toward the core — exactly the "repair inward" principle in action.

---

## 3. The Learner Model

The learner model is the system's representation of everything it knows about a specific person's understanding. It persists across sessions and evolves with every interaction.

### Learner Profile

The profile captures who the learner is and how they prefer to learn:

- **Identity**: Name, unique ID, creation date, session history.
- **Background skills**: A domain-specific skill map — for each relevant prerequisite area, a self-reported and (after assessment) verified skill level: *none*, *beginner*, *intermediate*, or *advanced*. In a biology domain, this might track chemistry background, lab experience, and math comfort. In a software domain, it might track programming language familiarity, framework experience, and systems knowledge.
- **Learning style**: How the learner prefers to engage — *concepts first* (explain the theory, then show examples), *examples first* (show concrete instances, then generalize), *challenges* (throw problems at me and I'll figure it out), or *mixed*.
- **Session statistics**: Total sessions, total time, longest streak of topics understood without detours.

### Node Status Model

Every node in the knowledge map (core, concept, detail, or foundation) has a status for this learner. There are **eight states** that form a lifecycle:

```
┌─────────────────┐
│ assumed_known    │  Learner self-reported knowing this.
│                  │  Unverified — could be accurate or not.
└────────┬────────┘
         │ (system verifies)
         ▼
┌─────────────────┐     ┌─────────────────┐
│ verified_known   │     │ shaky            │  Got it partially right.
│                  │     │                  │  Needs targeted follow-up.
│ Confirmed solid. │     └────────┬────────┘
└─────────────────┘              │
                                  ▼
                        ┌─────────────────┐
                        │ misconception    │  Confidently wrong.
                        │                  │  Highest priority to fix.
                        └────────┬────────┘
                                  │
         ┌────────────────────────┤
         ▼                        ▼
┌─────────────────┐     ┌─────────────────┐
│ identified_gap   │     │ being_taught     │  System is actively
│                  │     │                  │  teaching this now.
│ Confirmed they   │     └────────┬────────┘
│ don't know this. │              │
└────────┬────────┘              ▼
         │              ┌─────────────────┐
         └─────────────→│ taught_untested  │  Explained, but not
                        │                  │  yet verified.
                        └────────┬────────┘
                                  │ (system tests)
                                  ▼
                        ┌─────────────────┐
                        │ verified_learned │  Taught and confirmed.
                        │                  │  Node is now green.
                        └─────────────────┘
```

The most critical insight about these states: **`misconception` is more dangerous than `identified_gap`**. A learner who knows they don't know something will seek help. A learner who is confidently wrong will build further understanding on a broken foundation. The system always prioritizes surfacing and correcting misconceptions.

**Overlay Status: `needs_human_review`**

In addition to the eight primary states, a node can carry the `needs_human_review` overlay. This overlay does not replace the node's current status — it is applied on top of it. A node can be `shaky + needs_human_review` or `assumed_known + needs_human_review`. The overlay is set when the system's automated diagnostic and response analysis cannot classify the learner's understanding with confidence — for example, when the Response Analyzer receives ambiguous answers that resist classification even after follow-up attempts (see [Section 13d](#13d-ambiguous-learner-response)). When the overlay is active, the system continues the lesson while avoiding dependence on that node, and schedules re-assessment in the next session. The overlay is cleared when the node is successfully re-assessed or when a human reviewer provides input.

### Tracking Beliefs, Assumptions, Gaps, and Contradictions

For each topic, the system tracks four categories of learner cognition:

**Beliefs** — Things the learner has explicitly stated they believe. Each belief is recorded with:
- The claim itself ("I think enzymes only work at body temperature")
- Whether it's accurate
- The learner's apparent confidence
- Its current status: *active* (they still hold this belief), *revised* (they updated it), or *corrected* (the system corrected it)

**Assumptions** — Things the learner believes implicitly without realizing it. These are inferred from what they say, not from what they claim directly. For example, if a learner asks "which server does the Gateway forward the request to?" they're assuming a multi-server architecture — that assumption is recorded along with the evidence that revealed it.

**Gaps** — Concepts the learner hasn't encountered or hasn't understood. Each gap records what's missing, why it matters (which higher-level understanding it blocks), its importance level, and whether the system has addressed it.

**Contradictions** — When the learner says two things that conflict with each other. For example, saying "WebSockets are stateless" in one answer and "the server remembers which client sent each message" in another. The system records both statements, the tension between them, and whether the contradiction has been resolved.

### Confidence Decay and Spaced Repetition

A node verified as `verified_known` or `verified_learned` does not stay green forever. Confidence decays over time according to a formal decay model.

**Decay Function**

Each verified node carries a confidence value that decays exponentially:

```
confidence(t) = e^(-λ × t)
```

where `t` = days since last verification and `λ` (lambda) is the decay rate constant.

**Decay Rates by Verification Tier**

The tier at which a node was verified determines how quickly confidence fades:

| Tier | λ (decay rate) | Half-life | Meaning |
|------|---------------|-----------|---------|
| Recall | 0.099 | ~7 days | Memorized facts fade fast without reinforcement |
| Application | 0.033 | ~21 days | Applied knowledge persists longer |
| Synthesis | 0.014 | ~50 days | Deep understanding is the most durable |

**Re-verification Strengthens Memory**

Each time a node is explicitly re-verified (the system asks a question and the learner answers correctly), the decay rate permanently decreases by 20%, with a floor at 40% of the base rate:

```
λ = λ_base × max(0.4, 0.8^n)
```

where `n` = number of successful re-verifications. After 1 re-verification, λ drops to 80% of base. After 2, to 64%. After 4, it hits the floor at 40%. This models the well-established finding that spaced retrieval practice strengthens long-term retention.

**Re-verification Thresholds**

The system acts on decayed confidence at three thresholds:

| Confidence | Action |
|-----------|--------|
| ≤ 0.6 | Flag for **opportunistic re-check** — if the system is already asking questions nearby in the map, include a quick verification of this node |
| ≤ 0.4 | Schedule **active re-verification** — the system explicitly asks a targeted question at the start of the next session |
| ≤ 0.2 | Status **reverts to unverified** — the node loses its green status and returns to `assumed_known`, requiring full re-verification before dependents can be trusted |

Visually, the node's color shifts from bright green (confidence = 1.0) through yellow-green to yellow as confidence decays through these thresholds.

**Implicit Refresh Rule**

When a learner successfully uses a hard prerequisite while being tested on a dependent concept, the prerequisite's confidence resets to 1.0. For example, if a learner correctly applies knowledge of electron carriers while answering a synthesis question about oxidative phosphorylation, the electron carriers node resets to full confidence. However, implicit refresh does **not** reduce λ — only explicit re-verification (a direct question about the node itself) earns the 20% λ reduction. This distinction matters: implicit use confirms the knowledge is still accessible, but only deliberate retrieval practice strengthens the memory trace.

**Worked Example: Electron Carriers Node Over 55 Days**

```
Day 0:  Verified at Application tier. confidence = 1.0, λ = 0.033
Day 7:  confidence = e^(-0.033 × 7)  = e^(-0.231)  ≈ 0.79  (no action)
Day 14: confidence = e^(-0.033 × 14) = e^(-0.462)  ≈ 0.63  (no action)
Day 21: confidence = e^(-0.033 × 21) = e^(-0.693)  ≈ 0.50  (hit ≤0.6 → flagged
         for opportunistic re-check)
Day 25: Learner returns. System asks a targeted question about electron carriers.
         Learner answers correctly → re-verified.
         confidence resets to 1.0, n=1, λ = 0.033 × 0.8 = 0.0264
Day 40: confidence = e^(-0.0264 × 15) = e^(-0.396) ≈ 0.67  (no action — slower
         decay due to reduced λ)
Day 48: Learner answers a synthesis question about oxidative phosphorylation that
         requires using electron carrier knowledge → implicit refresh.
         confidence resets to 1.0 (but λ stays at 0.0264 — no reduction)
Day 55: confidence = e^(-0.0264 × 7) = e^(-0.185) ≈ 0.83  (still strong)
```

Without the re-verification on Day 25, confidence at Day 55 would have been `e^(-0.033 × 55) = e^(-1.815) ≈ 0.16` — below the revert threshold. The combination of explicit re-verification (reducing λ) and implicit refresh (resetting confidence) keeps the node green with minimal overhead.

This is **spaced repetition** built into the knowledge map itself, rather than bolted on as a separate flashcard system.

### Cognitive Snapshots

After every learner interaction (a response to a question, a question they ask, a confusion they express), the system produces a **cognitive snapshot** — a structured analysis of what just happened:

- New beliefs revealed
- New assumptions detected
- Gaps revealed or resolved
- Contradictions surfaced
- Whether the learner is ready to advance
- If they're stuck, the **missing link**: what specific prerequisite concept they lack and why it's blocking them
- The recommended next action: continue the lesson, probe deeper, start a detour, try a challenge, revisit an earlier topic, or celebrate and advance
- The single best next question to ask

These snapshots accumulate over time, forming a detailed cognitive history that agents can reference when making teaching decisions.

---

## 4. Content Ingestion — Teaching the System a New Domain

The system is designed to teach any domain, but it needs to be taught the domain first. Content ingestion is the process of transforming raw subject matter into a teachable knowledge map.

### The General Concept

A domain starts as unstructured knowledge — textbooks, documentation, expert explanations, reference materials. Content ingestion transforms this into the system's internal representation: a knowledge map with typed nodes, weighted dependencies, prerequisite chains, common misconceptions, and assessment questions.

The process works in layers:

1. **Topic identification**: The source material is analyzed to extract major topics (cores), supporting concepts, and atomic knowledge units (details). Cross-cutting prerequisites are identified as foundations. This is analogous to how a curriculum designer reads a textbook and creates a syllabus — but the output is a map, not a linear sequence.

2. **Dependency mapping**: For each concept, the system identifies what must be understood first. These prerequisites become directed dependencies in the map, each tagged with a dependency weight (hard, helpful, or optional).

3. **Misconception cataloging**: Common misunderstandings for each concept are recorded. These come from the source material itself ("students often confuse X with Y"), from expert annotation, or from patterns observed across learners over time. The misconception catalog is a living document — novel misconceptions are continuously discovered from learner data and propagated through a formal review pipeline (see [Section 12b](#12b-misconception-discovery-propagation)).

4. **Assessment design**: For each node, the system needs questions at three tiers — recall, application, and synthesis — that can verify whether a learner truly understands the concept.

### What a Domain Specification Needs

To onboard a new domain, the system requires:

- **Topics and concepts**: The subject matter broken into core, concept, detail, and foundation nodes. This can be provided explicitly by a domain expert or generated from source material with expert review.
- **Dependencies**: Which concepts require which prerequisites. At minimum, the hard dependencies must be specified; helpful and optional dependencies improve the experience but aren't strictly required.
- **Reference materials**: The actual content to teach from — textbook chapters, documentation pages, worked examples, diagrams. The system uses these to generate lessons, not to present them verbatim.
- **Known misconceptions**: The predictable ways learners misunderstand each concept. A good domain specification includes at least the top two or three misconceptions per major concept.
- **Assessment questions**: Sample questions at different tiers, or enough source material for the system to generate them.

### Automated Construction vs. Expert Curation

In practice, domain onboarding is a collaboration between automated map construction and human expertise:

- **Automated**: An AI agent can analyze source material and propose a map structure — identifying topics, suggesting hierarchies, inferring dependencies from the order concepts are introduced in reference material.
- **Expert-curated**: A domain expert reviews the proposed map, corrects dependency weights, adds misconceptions the AI wouldn't know about, and validates that the detail-level decomposition matches how the subject is actually understood (not just how textbooks organize it).

The quality of the domain specification directly determines the quality of the diagnostic and teaching experience. A poorly mapped dependency structure leads to the system teaching concepts before their prerequisites are solid, or missing the real reason a learner is confused.

### Example: Onboarding a Non-Technical Domain

Consider onboarding "Culinary Fundamentals" as a domain:

- **Cores**: Knife Skills, Heat and Cooking Methods, Flavor Building, Sauce Foundations, Baking Principles.
- **Concepts under Heat and Cooking Methods**: Conduction (pan-based), Convection (oven-based), Radiation (broiling/grilling), Moist Heat (braising, steaming), combination methods.
- **Details under Conduction**: What thermal conductivity means, how pan material affects cooking, the Maillard reaction, oil smoke points.
- **Foundation**: "Heat transfer principles" — supports Conduction, Convection, and Radiation across multiple concepts.
- **Hard dependency**: Understanding the Maillard reaction requires understanding what thermal conductivity means (you can't explain *why* stainless steel sears differently from cast iron without it).
- **Known misconception**: "Searing locks in juices" — a persistent misconception that the system would flag and specifically design assessment questions around.

The pattern is identical to a software or biology domain: decompose into map, weight dependencies, catalog misconceptions, design assessments. The domain changes; the structure doesn't.

---

## 5. The Diagnostic Engine

The diagnostic engine is the system's core intellectual capability. It determines what a learner knows, what they don't know, and — most importantly — what they *think* they know but have wrong.

### Strategic Testing, Not Exhaustive Testing

The diagnostic engine operates like a **binary search through the knowledge map**, not a linear scan. It doesn't test every node. It picks the most **leveraged** nodes — nodes that, if wrong, would mean many other nodes are also wrong — and tests those first.

For example: if a learner self-reports "intermediate" knowledge of cellular respiration, the system doesn't quiz them on every detail. It picks one question that an intermediate learner would definitely know — say, "What molecule carries electrons from the Krebs cycle to the electron transport chain?" If they know it's NADH, they probably understand the overall flow. If they don't, a large portion of their claimed knowledge is suspect.

One well-chosen question can verify or invalidate an entire concept.

### Phase 1: Self-Report Assessment

The diagnostic process begins with self-report. The learner rates their familiarity with each major area of the domain:

- "How familiar are you with harmonic theory?" → None / Beginner / Intermediate / Advanced
- "Have you ever worked with relational databases?" → etc.

Self-report sets all nodes within each area to `assumed_known` at the reported level. But self-reports are unreliable in predictable ways: beginners overestimate (Dunning-Kruger), experts underestimate (curse of knowledge), and everyone has blind spots. Self-report is just the starting point — a rough map to be verified.

### Phase 2: Targeted Verification

This is the smart part. The system selects a small number of **strategically chosen verification questions** to test the self-reported levels.

The selection strategy:
- Pick nodes with the most downstream dependents (high leverage)
- Pick nodes at the boundary of claimed knowledge (where "I know this" meets "I don't know that")
- Pick nodes where misconceptions are common (high value to test)

Each verification question is designed so that different wrong answers point to different diagnoses:

- **Correct answer** → This concept is probably solid. The system can trust the self-report for this area and move on.
- **Partially correct** → The learner has the right intuition but shaky details. Mark as `shaky` and plan targeted follow-up.
- **Confidently wrong** → Misconception detected. This is the most valuable diagnostic outcome — the system now knows exactly what to prioritize.
- **"I don't know"** → Honest gap. The self-report was too optimistic. Adjust the entire concept downward.

Typically, 5–10 well-chosen questions can map a learner's actual level across an entire domain. The system doesn't need to ask more because each question is chosen to maximize information gain.

### Phase 3: Map Coloring

After assessment, every node in the knowledge map receives a color based on its status:

```
  🟢  Green    — Verified known (passed application-level test)
  🟡  Yellow   — Assumed known (self-reported, unverified)
  🔴  Red      — Identified gap (confirmed they don't know this)
  🟠  Orange   — Misconception (they think they know it but they're wrong)
  ⚪  Gray     — Unknown (hasn't been assessed yet)
```

The learner sees this map and can immediately understand the shape of their knowledge: "I'm strong in the center-left region but weak on the right." This is their **knowledge map** — a visual representation of exactly where they are.

### The Diagnostic Narrowing Algorithm

When the system detects confusion or incorrect responses during teaching, it runs a focused diagnostic to locate the exact source. The diagnostic direction is **narrow outward** — start at the core level and probe outward through concepts to details until the gap is found. This follows a six-step narrowing pattern:

```
1. OBSERVE
   What did the learner say or ask that signals confusion?
   What specific words or claims triggered this diagnosis?

2. HYPOTHESIZE
   Generate 2–3 hypotheses for WHERE the gap might be.
   "They might not understand Concept A, or they might be confusing
   Concept B with C, or they might be missing a Foundation."

3. DISCRIMINATE
   Ask ONE question that distinguishes between hypotheses.
   A good discriminating question eliminates at least half the
   hypotheses with a single answer.

4. NARROW
   Based on their answer, eliminate hypotheses. Narrow outward
   into the remaining concept → its sub-concepts → its details.

5. LOCATE
   After 2–3 questions, identify the exact detail or foundation
   that's broken.

6. VERIFY
   "So the thing that's unclear is [specific detail]?"
   Let the learner confirm before teaching. Sometimes naming
   the confusion is enough to resolve it.
```

**Diagnostic direction**: Narrow outward (Core → Concept → Detail).
**Teaching direction**: Repair inward (Detail/Foundation → Concept → Core).

Three questions should locate any gap in a map of variable depth. The key is asking **discriminating questions** — questions where different answers point to different diagnoses. A question that only confirms one hypothesis wastes a turn; a question that distinguishes between two or three hypotheses cuts the search space in half.

#### Formal Specification: Diagnostic Narrowing

The six-step narrative above provides the high-level overview. Below is the formal algorithm the system executes.

**Step 1: Hypothesis Generation**

When a confusion signal is detected (incorrect answer, expressed confusion, or contradictory statement), the system generates a ranked list of suspect nodes:

1. **Collect candidates**: Gather all prerequisite nodes of the current topic that are not `verified_known` or `verified_learned`.
2. **Filter by relevance**: Discard candidates that have no semantic connection to the confusion signal. (For example, if the learner's error involves energy transfer, prerequisites about cell structure are filtered out.)
3. **Rank by prior probability**:
   - `shaky` nodes → highest prior (known to be weak)
   - `assumed_known` nodes → high prior (unverified, may be wrong)
   - Decayed nodes (confidence ≤ 0.6) → medium prior (once known, now uncertain)
   - `foundation` nodes → elevated prior within their rank (cross-cutting gaps are more likely to cause confusion in dependent topics)

This produces an ordered hypothesis list of 2–5 candidate nodes, each representing a possible root cause of the confusion.

**Step 2: Question Selection via Information Gain**

For each candidate question `q`, the system predicts the probability distribution over observable outcomes `O = {correct, incorrect_with_specific_error, partial, confused, "I don't know"}` under each hypothesis, then calculates the information gain:

```
IG(q) = -Σ P(O_i) × log₂(P(O_i))
```

where `P(O_i)` is the marginal probability of outcome `i` across all hypotheses. The question with the highest IG is selected.

A perfect discriminating question — one where each hypothesis predicts a different outcome — yields `IG = log₂(|hypotheses|)`. For 3 hypotheses, the maximum IG is `log₂(3) ≈ 1.58 bits`. In practice, the system targets questions with IG ≥ 1.0 bit, accepting imperfect discrimination when no better question is available.

**Step 3: Ambiguous Answer Handling**

After the learner responds, the system classifies the response into one of five categories:

| Response Type | Interpretation | Action |
|--------------|---------------|--------|
| Correct | This hypothesis is eliminated | Remove from candidate list, narrow remaining |
| Incorrect with identifiable error | Error signature matches a specific hypothesis | Confirm that hypothesis, proceed to locate exact gap |
| Partial | Gap is deeper than the hypothesis level | Expand the hypothesis node into its children, re-rank |
| Confused / incoherent | Gap is more foundational than hypothesized | Regenerate hypotheses one level deeper in the prerequisite chain |
| "I don't know" | Honest gap confirmed at this node | Mark as `identified_gap`, begin teaching |

**Step 4: Variable Depth Scaling**

Diagnostic narrowing operates at progressively finer granularity:

- **Depth 1 — Concept level**: Initial hypotheses target concepts (e.g., "The learner may not understand the Krebs cycle"). Budget: 1–2 questions.
- **Depth 2 — Sub-concept level**: When a concept is confirmed as the problem area, expand into its children (e.g., "Within the Krebs cycle, is the issue with Acetyl-CoA entry, electron carrier production, or CO₂ release?"). Budget: 2–3 questions.
- **Depth 3+ — Detail level**: When a sub-concept is confirmed, expand into its details (e.g., "Within electron carrier production, is the issue with what NADH is, how FADH₂ differs, or where the electrons come from?"). Budget: 3–5 questions.

The **total question budget is 4 questions** across all depths. If the algorithm exhausts this budget without achieving >0.8 confidence in the gap location, the error recovery protocol is triggered (see [Section 13a](#13a-diagnostic-failure)).

**Worked Example: Learner Studying Cellular Respiration**

A learner is studying the core topic "How Cellular Respiration Works." They give a response about the overall process that correctly describes glycolysis but completely ignores the Krebs cycle and oxidative phosphorylation, instead jumping from "pyruvate is made" to "ATP is produced."

```
OBSERVE: Learner's response skips Krebs cycle and oxidative phosphorylation.
         Confusion signal: missing two of three major stages.

HYPOTHESIS GENERATION:
  Candidates (prerequisites of cellular respiration, not verified):
    H1: Krebs Cycle concept (status: assumed_known) — prior: 0.40
    H2: Oxidative Phosphorylation concept (status: assumed_known) — prior: 0.35
    H3: Electron carriers detail (status: shaky) — prior: 0.50
  Ranked: H3 (0.50), H1 (0.40), H2 (0.35)

QUESTION 1: "When pyruvate enters the mitochondria, what molecules carry
             the energy extracted from it to the next stage?"
  IG calculation:
    If H3 (electron carriers gap): P(incorrect) = 0.8, P(partial) = 0.2
    If H1 (Krebs cycle gap): P(partial) = 0.6, P(correct) = 0.3, P(confused) = 0.1
    If H2 (ox-phos gap): P(correct) = 0.5, P(partial) = 0.4, P(incorrect) = 0.1
    Marginal: P(correct)=0.25, P(incorrect)=0.31, P(partial)=0.38, P(confused)=0.06
    IG = -(0.25×log₂0.25 + 0.31×log₂0.31 + 0.38×log₂0.38 + 0.06×log₂0.06)
       ≈ 1.68 bits (near-maximum discrimination)
  Selected: This question.

  Learner responds: "I think... NADH? But I'm not really sure what it does."
  Classification: Partial — knows the name but not the function.
  Update: H3 confirmed as a real gap, but response suggests the gap is
          at a specific detail level, not the entire concept.

NARROW (Depth 2): Expand electron carriers into sub-details:
    H3a: What NADH is (molecular identity)
    H3b: How NADH carries electrons (mechanism)
    H3c: Where NADH delivers electrons (destination = ETC)

QUESTION 2: "You mentioned NADH — what does NADH actually carry, and
             where does it deliver its cargo?"
  Learner responds: "It carries... energy? To make ATP somehow?"
  Classification: Incorrect with identifiable error — the learner
    thinks NADH carries "energy" generically rather than specific
    electrons, and doesn't know the destination.
  Gap located: H3b (how NADH carries electrons) AND H3c (destination).
  Confidence: 0.85 — the gap is the mechanism and destination of
    electron carriers, specifically chemiosmosis.

VERIFY: "So the piece we need to fill in is: what exactly electron
         carriers like NADH transport, and what happens when they
         deliver it. Does that match where you feel uncertain?"
  Learner: "Yes, exactly — I know NADH is important but I don't
            understand what it actually does."

RESULT: Gap located in 2 questions + 1 verification.
        Target for teaching: chemiosmosis and the electron transport
        chain, starting from what electrons are and why moving them
        releases energy.
```

### Misconception Detection via Transfer Questions

Misconceptions are the most dangerous state and the hardest to detect. A learner with a misconception answers questions **confidently** but **incorrectly** — and may even answer certain questions correctly by accident if they've memorized surface-level patterns.

The system detects misconceptions using **transfer questions**: after a learner answers a question correctly in one context, it asks the same underlying concept in a **different** context. If they can apply the concept in a new setting, they truly understand it. If they can't, they may have memorized the answer rather than grasped the principle.

**Example (music domain)**:
- Question: "In the key of C major, what chord typically comes before the tonic?" → "G major (the dominant)" ✅
- Transfer: "You're composing in E♭ major. What chord creates the strongest pull toward resolution?" → If they can answer B♭ major, they understand the dominant-tonic relationship as a *principle*. If they can only answer for C major, they memorized a fact.

**Example (science domain)**:
- Question: "Why does ice float in water?" → "Because ice is less dense than liquid water" ✅
- Transfer: "Most substances are denser as solids than as liquids. Why is water unusual?" → If they can explain hydrogen bonding and crystal structure, they understand the *why*. If they just memorized "ice is less dense," the transfer exposes the gap.

### Three Assessment Tiers

The system uses three tiers of assessment, each testing progressively deeper understanding:

**Tier 1 — Recall**: "What is X?" Tests whether the learner can state the concept. This is the weakest form of verification. Passing recall only gets a node to a provisional status — it confirms vocabulary but not understanding.

**Tier 2 — Application**: "Given this scenario, what happens?" Tests whether the learner can *use* the concept in a specific situation. This is the primary verification standard — a node should only go fully green after passing an application-level test.

**Tier 3 — Synthesis**: "How does X relate to Y?" or "What would break if X were different?" Tests whether the learner can connect the concept to other knowledge, reason about it abstractly, and predict consequences. This reveals deep understanding versus surface memorization.

A node verified at the synthesis level decays more slowly than one verified only at recall. The tier of verification determines both the confidence level and its persistence over time.

---

## 6. The Teaching Engine

The teaching engine takes the diagnostic engine's output — the map of what the learner knows and doesn't know — and determines *what* to teach, *in what order*, and *how* to present it.

### Strategy Rules

The teaching engine follows strict rules that prevent common tutoring failures:

**Rule 1: Never teach a core if a concept is red.**
If "How Cellular Respiration Works" depends on "Glycolysis" and the Glycolysis concept is red, teach Glycolysis first. If Glycolysis depends on "What glucose is" and that detail is red, teach glucose first. Always repair inward: fix details/foundations → fix concepts → synthesize core.

**Rule 2: Teach the highest-impact gap first.**
A red detail that blocks THREE cores is more important than a red detail that blocks one. The system calculates which gaps have the most downstream dependencies and prioritizes those using a formal **impact score** algorithm (see [Impact-Ordered Prioritization](#impact-ordered-prioritization) below for the full specification). Foundations naturally score highest because they support nodes across multiple regions of the map.

**Rule 3: Match the representation to the knowledge type.**
Different types of knowledge require different teaching formats:

| Knowledge Type | Best Representation | Example |
|---|---|---|
| Structure / hierarchy | Diagram that builds itself | Org charts, system architecture, taxonomies |
| Process / sequence | Step-by-step flow trace | How a request flows through a system, metabolic pathways |
| Comparison / contrast | Side-by-side with differences highlighted | TCP vs. UDP, mitosis vs. meiosis, major vs. minor keys |
| Cause and effect | Animated arrows showing "if X then Y" | Why searing creates browning, why recursion needs a base case |
| Definition / terminology | Brief text + concrete example | What an enzyme is, what a closure is, what a fermata means |
| Pattern / principle | Multiple examples collapsing into one pattern | Design patterns, chemical reaction types, chord progressions |
| Spatial / physical | Annotated diagram or animation | Cell structure, circuit layout, guitar fretboard |

The system selects the representation based on what's being taught, not a one-size-fits-all approach. A lesson about a process uses flow traces. A lesson about comparison uses side-by-side views. This is automatic — the lesson generator selects the format based on the knowledge type tag of each node.

**Rule 4: Test after teaching, at the right level.**
If you taught a detail, test the detail. Then test whether the concept now makes sense. Then test whether the core now makes sense. Don't skip levels. Each level of testing ensures that the repair propagated correctly inward through the map.

### The Repair-Inward Sequence

When the system identifies a gap, it follows a strict repair sequence:

```
1. LOCATE the broken detail or foundation
   (via the diagnostic narrowing algorithm)

2. TEACH the detail/foundation
   (using the appropriate representation for its knowledge type)

3. TEST the detail/foundation
   (application-level question, not just recall)

4. If detail passes:
   RECONNECT to the concept
   "Now that you understand [detail], here's how it fits
   into [concept]..."

5. TEST the concept
   (can they use the concept in a scenario?)

6. If concept passes:
   RECONNECT to the core
   "With [concept] solid, let's revisit [core]..."

7. TEST the core
   (synthesis-level question connecting multiple concepts)

8. If core passes:
   Mark as verified, advance to next topic
```

This repair-inward approach prevents a common failure mode: teaching a concept, seeing the learner nod, and moving on — only to discover later that they understood the explanation in isolation but can't connect it to the larger picture.

### Impact-Ordered Prioritization

When multiple gaps exist (which is common, especially after initial assessment), the system prioritizes by downstream impact using a formal scoring algorithm.

**Definition**: The impact score of a gap node is the weighted count of distinct unlearned nodes reachable from it via dependency edges in the knowledge DAG.

**Algorithm** (BFS traversal):

```
function impact_score(gap_node, knowledge_map):
    visited = {gap_node}
    queue = [gap_node]
    score = 0.0

    while queue is not empty:
        current = queue.pop_front()
        for each (dependent, weight) in current.dependents:
            if dependent not in visited:
                visited.add(dependent)
                if dependent.status not in {verified_known, verified_learned}:
                    if weight == "hard":
                        score += 1.0
                    else if weight == "helpful":
                        score += 0.5
                    // optional dependencies contribute 0 — they don't block
                    queue.append(dependent)

    // Misconception multiplier (Design Principle 5):
    if gap_node.status == "misconception":
        score *= 1.5

    return score
```

**Key properties**:
- Uses a visited set to prevent double-counting in DAG fan-out (a node reachable via two paths is counted once)
- Hard dependencies contribute 1.0 per blocked node; helpful dependencies contribute 0.5
- Optional dependencies contribute 0 — they don't block understanding
- Only nodes that are NOT already `verified_known` or `verified_learned` are counted (fixing a gap that unblocks already-known nodes is less valuable)
- Misconceptions receive a 1.5× multiplier because they silently corrupt understanding of everything built on top of them (Design Principle 5)
- **Tie-breaking**: When two gaps have equal impact scores, prefer the gap with greater average depth from center (more peripheral = more foundational = should be taught first per "repair inward" principle)

**Worked Example: Three Gaps in a Biology Map**

After initial assessment, the system identifies three gaps:

```
Gap 1: "ATP as energy currency" (foundation)
  Status: identified_gap
  Dependents via hard edges:
    → Glycolysis (not verified) → Net energy yield (not verified)     = 2 nodes
    → Active transport (not verified)                                  = 1 node
    → Krebs Cycle (not verified) → Acetyl-CoA (not verified)          = 2 nodes
  Dependents via helpful edges:
    → Protein synthesis (not verified)                                 = 0.5
  Score = (2 + 1 + 2) × 1.0 + 0.5 = 5.5

Gap 2: "Electron carriers (NADH, FADH2)" (detail)
  Status: misconception (learner thinks NADH "creates" energy)
  Dependents via hard edges:
    → Oxidative phosphorylation (not verified)                         = 1 node
    → Chemiosmosis (not verified)                                      = 1 node
    → Oxygen as final acceptor (not verified)                          = 1 node
  Score = 3.0 × 1.5 (misconception multiplier) = 4.5

Gap 3: "Lock-and-key model" (detail under enzyme catalysis)
  Status: identified_gap
  Dependents via hard edges:
    → Activation energy (verified_learned — skip)                      = 0 nodes
  Dependents via helpful edges:
    → Glycolysis (not verified)                                        = 0.5
  Score = 0 + 0.5 = 0.5

Priority order: Gap 1 (5.5) → Gap 2 (4.5) → Gap 3 (0.5)
```

The system teaches ATP first because it unblocks the most downstream understanding. Electron carriers come second despite being a misconception (higher per-node urgency) because ATP unblocks more total learning paths. Lock-and-key is last — it's a nice-to-know detail with minimal downstream impact.

This means the system naturally gravitates toward foundations and widely-shared details — because these, by definition, have the most downstream dependents. It also means that fixing one high-impact gap often unblocks several teaching paths at once.

### The Detour System

A **detour** occurs when, during a lesson, the system discovers that the learner is missing a prerequisite that wasn't caught during assessment. The detour system handles this gracefully:

1. The current lesson is **paused**. The system saves the exact position (the checkpoint) and the context of what was being taught.
2. The system identifies the **missing prerequisite** — the specific detail or foundation that the learner needs.
3. A **mini-lesson** on the prerequisite is generated and taught. This mini-lesson is short (5–10 steps), focused on a single concept, and ends with a quick verification question.
4. When the prerequisite is verified, the system **pops back** to the original lesson, replaying the last step before the interruption for context, and continues.

Detours can **nest** — a detour can trigger its own detour if the prerequisite itself has a prerequisite. The system enforces a **maximum nesting depth of 3** to prevent infinite regression. When nesting hits this limit, the error recovery protocol takes over — popping the detour stack, running a focused foundation assessment, and either teaching the foundations directly or recommending a prerequisite learning path (see [Section 13b](#13b-detour-depth-exceeded) for the full recovery procedure).

The detour stack works like a call stack in programming:

```
Teaching: Cellular Respiration (core)
  └─ Detour: Electron Transport Chain (concept was shaky)
       └─ Detour: What is an electron carrier? (missing detail)
            Teach → Verify → Pop back to Electron Transport Chain
       Resume → Verify → Pop back to Cellular Respiration
  Resume from checkpoint
```

### Interrupt-Driven Learning

The learner can trigger a diagnostic pause at **any point** during a lesson. They don't have to wait for a question — they can say "I'm confused" or "what does that mean?" at any moment.

When an interrupt occurs:

1. The system first determines the **scope** of the confusion:
   - Can the learner identify which concept is confusing? → Drill into that concept's details.
   - Can they identify which detail? → Diagnose that specific detail.
   - Can't identify anything specific? → They're lost at the core level. Back up further.

2. The system runs the diagnostic narrowing algorithm to locate the exact gap.

3. Based on the diagnosis, it either:
   - Provides a **quick clarification** (if the issue is just unfamiliar terminology or unclear wording)
   - Starts a **detour** (if there's a genuine missing prerequisite)
   - Offers a **simpler example** (if the concept is right but the presentation was too abstract)

The interrupt system turns learning from a passive experience ("keep watching and hope you follow") into an active conversation ("stop me anytime and I'll figure out exactly what you need").

---

## 7. Agent Architecture

The system is powered by **specialized AI agents**, each with a distinct cognitive function. No single agent tries to do everything — instead, each agent is an expert in one aspect of the tutoring process, and they collaborate by passing structured data between each other.

### Overview

```
┌─────────────────────────────────────────────────────────┐
│                   AGENT ARCHITECTURE                     │
│                                                          │
│  ┌──────────┐   ┌──────────┐   ┌──────────────────┐    │
│  │Assessment │──→│ Overview  │──→│ Learning Path    │    │
│  │  Agent    │   │ Generator │   │ Generator        │    │
│  └──────────┘   └──────────┘   └────────┬─────────┘    │
│                                          │               │
│                                          ▼               │
│                                 ┌──────────────────┐    │
│                                 │ Topic Selector   │    │
│                                 └────────┬─────────┘    │
│                                          │               │
│       ┌──────────────────────────────────┤               │
│       │                                  │               │
│       ▼                                  ▼               │
│  ┌──────────┐                   ┌──────────────────┐    │
│  │ Lesson   │                   │  Quick            │    │
│  │Generator │                   │  Clarification    │    │
│  └────┬─────┘                   │  Handler          │    │
│       │                         └──────────────────┘    │
│       ▼ (learner responds)                               │
│  ┌──────────┐    gap found     ┌──────────────────┐    │
│  │ Response │───────────────→  │  Confusion        │    │
│  │ Analyzer │                  │  Analyzer         │    │
│  └──────────┘                  └──────────────────┘    │
│       │                                                  │
│       ▼                                                  │
│  (updates learner model, loops back to Lesson Generator  │
│   or Topic Selector as needed)                           │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Agent 1: Assessment Agent

**Role**: Verifies the learner's self-reported background by asking targeted diagnostic questions.

**When invoked**: Once, during onboarding, after the learner fills in their background skill levels.

**Inputs**: The learner's self-reported skill levels across all relevant areas of the domain.

**What it does**: Generates 2–3 conversational questions per skill area, designed to verify actual competence without feeling like a quiz. For a learner who self-reports "intermediate" on a topic, it picks a question that an intermediate person would definitely know. For a beginner, it asks about adjacent experience that might transfer.

**Outputs**: Adjusted skill levels (which may differ from self-report), assessment notes explaining what was observed, and a suggested starting point (complete beginner, has foundation, or experienced).

### Agent 2: Overview Generator

**Role**: Creates a calibrated high-level overview of the entire domain, personalized to the learner's assessed level.

**When invoked**: Once, immediately after assessment. This is the first substantive content the learner sees.

**Inputs**: The learner's assessed level, adjusted background, and assessment notes.

**What it does**: Generates a complete overview covering the domain's structure, major components, and how they relate to each other. Critically, it **calibrates** the explanation: a complete beginner gets everyday analogies and no jargon; someone with a foundation gets technical language with domain-specific terms explained; an expert gets concise architectural descriptions.

**Outputs**: A sequence of presentation steps (text explanations, diagrams, flow traces) that give the learner a mental model of the entire domain before diving into any part. The goal: after this overview, the learner should be able to sketch the domain's high-level structure from memory.

### Agent 3: Learning Path Generator

**Role**: Creates a personalized learning path based on the learner's gaps, strengths, and the domain's dependency structure.

**When invoked**: Once, after the overview. May be re-invoked if the learner's assessed level changes significantly during teaching.

**Inputs**: The learner's profile, adjusted background, assessment notes, and the full domain topic list with dependencies.

**What it does**: Categorizes every topic in the domain into one of three buckets:
- **Recommended**: Topics the learner should study, ordered by priority (start here → core → deep → advanced)
- **Skippable**: Topics their background already covers (only if the system is confident)
- **Deferred**: Topics too advanced for now, to be revisited after building more foundation

For each topic, it provides a reason for the categorization and any pacing adjustments (slower for areas they'll struggle with, faster for areas where they have adjacent experience).

**Outputs**: An ordered learning path with rationale, plus a personalized opening message that helps the learner understand why the path is structured this way.

### Agent 4: Lesson Generator

**Role**: Creates lesson content for a specific topic, personalized to the learner's current knowledge state.

**When invoked**: Each time the learner starts a new topic.

**Inputs**: The learner's full profile, topics already completed, previous struggles (detours), existing beliefs/assumptions/gaps for this topic, the topic's description and key concepts/details, common misconceptions for this topic, and reference material.

**What it does**: Generates a lesson as a sequence of teaching steps, following these rules:
- Start by connecting to what the learner already knows from completed topics
- Build incrementally — each step adds one new idea
- After every 3–4 teaching steps, include a question that tests understanding (not recall)
- Design questions that reveal hidden assumptions
- Include checkpoints throughout (for resume-after-detour)
- If the learner has known misconceptions, design steps that surface and correct them
- End with a summary and celebration of what they now understand

**Outputs**: An ordered sequence of presentation steps (text, diagrams, code examples, questions, checkpoints) that form a complete lesson.

> **Error recovery**: All agents operate under the error recovery protocols defined in [Section 13](#13-error-recovery-and-graceful-degradation). When any agent produces incoherent, off-topic, or malformed output, the system detects the failure, retries with a modified prompt, and falls back to template responses if necessary.

### Agent 5: Confusion Analyzer

**Role**: Diagnoses **why** a learner is confused — specifically, identifies the missing prerequisite concept that's blocking their understanding.

**When invoked**: Whenever the learner expresses confusion, asks a question mid-lesson, or gives a response that reveals a gap.

**Inputs**: The current topic, what was just being taught, the learner's question or confused response, their background, their completed topics, and their known gaps.

**What it does**: The key distinction — this agent's job is **not** to answer the learner's question directly. Its job is to figure out what prerequisite concept the learner is missing that *makes* the current topic confusing. It classifies the confusion into one of three types:
- **Missing concept**: A genuine prerequisite gap that requires a detour
- **Needs clarification**: The concept is understood but the wording or example was unclear
- **Needs simpler example**: The concept was presented too abstractly

**Outputs**: For missing concepts, the specific prerequisite, why it's blocking, a brief explanation, and how to connect it back to the original topic. For clarification or simpler examples, the re-explanation itself.

### Agent 6: Response Analyzer

**Role**: Analyzes what the learner said after being asked a question, extracts cognitive insights, and updates the knowledge model.

**When invoked**: After every learner response to a question during a lesson.

**Inputs**: The topic being taught, the question that was asked, the learner's response, and their current knowledge model (beliefs, assumptions, gaps).

**What it does**: Carefully analyzes the response for:
- What they said that's correct (even partially)
- What reveals a misconception
- What they didn't say that someone with understanding would say
- What they're assuming without realizing it
- Whether they're ready to advance

**Outputs**: New beliefs, revealed assumptions, discovered gaps, resolved gaps, contradictions, whether the learner is ready to advance, specific encouragement (not generic — "Right, you noticed that the dominant chord creates tension, which is exactly the point" rather than "Great job!"), gentle corrections for misconceptions, and the recommended next action (continue / probe deeper / detour / challenge). When the Response Analyzer cannot classify a learner's response with confidence (ambiguous, off-topic, or contradictory in ways that resist interpretation), the ambiguous response recovery protocol is triggered (see [Section 13d](#13d-ambiguous-learner-response)).

### Additional Agents

**Topic Selector** — Decides what to teach next when the learner completes a topic or starts a new session. It considers prerequisite satisfaction, cross-topic gaps that need revisiting, engagement patterns (switching topics if the learner has been on one area too long), and the path that maximizes overall understanding across the map.

**Quick Clarification Handler** — Handles simple mid-lesson questions that don't require a full detour. If the learner asks "what does this term mean?" or "why this specific notation?", this agent provides a 1–3 sentence answer. If it determines the question reveals a deeper confusion, it escalates to the Confusion Analyzer.

### How Data Flows Between Agents

The agents form a pipeline where each agent's output becomes the next agent's input:

```
Learner fills background ──→ Assessment Agent ──→ adjusted levels
                                                       │
adjusted levels ──→ Overview Generator ──→ overview content
                                                       │
adjusted levels + assessment ──→ Learning Path Generator ──→ ordered path
                                                                 │
                    ┌────────────────────────────────────────────┘
                    ▼
            Topic Selector ──→ next topic ID
                    │
                    ▼
     topic + learner state ──→ Lesson Generator ──→ lesson steps
                                                        │
                         learner responds to question ◄──┘
                                    │
                                    ▼
                            Response Analyzer ──→ cognitive snapshot
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
              (continue)    (probe deeper)    (start detour)
                    │               │               │
                    │               │               ▼
                    │               │       Confusion Analyzer ──→ missing link
                    │               │               │
                    │               │               ▼
                    │               │       Lesson Generator (mini-lesson)
                    │               │               │
                    ▼               ▼               ▼
              ┌─────────────────────────────────────────┐
              │         Update Learner Model             │
              │    Loop back to Lesson Generator or      │
              │    Topic Selector as appropriate          │
              └─────────────────────────────────────────┘
```

The learner model is the shared state that all agents read from and write to. It serves as the system's memory — every agent's decisions are informed by the cumulative history of what every other agent has observed.

---

## 8. The End-to-End Workflow

### Complete Lifecycle

A learner's journey through the system follows this path:

```
┌───────────┐     ┌────────────┐     ┌──────────┐     ┌──────────────┐
│   New      │────→│ Onboarding │────→│Assessment│────→│   Overview   │
│  Learner   │     │  (profile) │     │ (verify) │     │  (big picture│
└───────────┘     └────────────┘     └──────────┘     └──────┬───────┘
                                                              │
                                                              ▼
┌───────────┐     ┌────────────┐     ┌──────────────────────────────┐
│  Mastery   │◄───│  Advance   │◄────│       Learning Path          │
│ (complete) │    │  or Loop   │     │   (personalized path)        │
└───────────┘    └────────────┘     └──────────────┬───────────────┘
                       ▲                            │
                       │                            ▼
                       │              ┌──────────────────────────────┐
                       │              │   Teach / Diagnose Cycle     │
                       │              │                              │
                       │              │  Lesson ──→ Question ──→     │
                       │              │  Response Analysis ──→       │
                       │              │  (Continue|Probe|Detour|     │
                       └──────────────│   Advance)                   │
                                      │                              │
                                      └──────────────────────────────┘
```

**Step 1 — Onboarding**: The learner creates a profile. They provide their name and self-report their background skill levels across the domain's prerequisite areas.

**Step 2 — Assessment**: The Assessment Agent asks 2–3 targeted questions per skill area to verify the self-report. The result: adjusted skill levels and an overall starting point classification.

**Step 3 — Overview**: The Overview Generator creates a personalized high-level tour of the entire domain. This gives the learner a mental framework before they dive into specifics. After the overview, the learner should be able to describe the domain's major components and how they connect.

**Step 4 — Learning Path**: The Learning Path Generator produces a personalized learning path, showing the learner which topics to study, in what order, and why. The learner can see their path and understands the rationale.

**Step 5 — Teach/Diagnose Cycle**: This is where the learner spends most of their time. The cycle repeats for each topic:

1. The **Lesson Generator** creates content for the current topic
2. The lesson plays, interspersed with questions
3. The learner responds to questions
4. The **Response Analyzer** interprets each response, updating the learner model
5. Based on the analysis, the system decides:
   - **Continue**: Understanding is solid, proceed to the next lesson step
   - **Probe deeper**: Understanding seems shaky, ask a more targeted question
   - **Detour**: A missing prerequisite is identified — pause the lesson, teach the prerequisite, return
   - **Advance**: The topic is mastered — celebrate, update the map, and move to the next topic

**Step 6 — Advance or Loop**: When a topic is complete, the Topic Selector chooses the next topic based on the current state of the knowledge map, and the cycle continues.

**Step 7 — Mastery**: Over time, the knowledge map fills in. All cores are green. The learner has a verified, comprehensive understanding of the domain.

### Session Resumption

Learning doesn't happen in a single sitting. The system is designed for **weeks-long journeys** with sessions that might be days or weeks apart.

When a returning learner opens a new session:

1. **Load persistent state**: The learner model, knowledge map status, curriculum position, and compressed conversation history are loaded.

2. **Apply confidence decay**: Nodes that were verified in previous sessions but haven't been touched since are decayed based on elapsed time. Nodes verified at higher assessment tiers decay more slowly.

3. **Quick re-assessment**: For nodes that have decayed past a threshold, the system asks 1–2 quick verification questions. This takes under a minute and ensures the learner hasn't forgotten critical foundations.

4. **Re-contextualize**: The system summarizes where the learner left off: "Last time, we covered [topic]. You had just demonstrated understanding of [concept] and we were about to move into [next concept]. Does that sound right?" This helps the learner mentally re-engage.

5. **Resume or redirect**: If the re-assessment reveals that previously solid foundations are now shaky, the system adjusts the plan. Otherwise, it picks up exactly where the learner left off.

### The Teach/Diagnose Loop in Detail

The inner loop of a single lesson looks like this:

```
┌─────────────────────────────────────────────────────┐
│                  TEACH/DIAGNOSE LOOP                  │
│                                                       │
│  ┌───────────┐                                       │
│  │ Present   │  (text, diagram, code, etc.)          │
│  │ lesson    │                                       │
│  │ step      │                                       │
│  └─────┬─────┘                                       │
│        │                                              │
│        ▼                                              │
│  ┌───────────┐         ┌───────────────────────┐     │
│  │ Ask       │────────→│ Learner responds      │     │
│  │ question  │         │ or asks question      │     │
│  └───────────┘         └──────────┬────────────┘     │
│                                    │                   │
│                                    ▼                   │
│                        ┌───────────────────────┐      │
│                        │ Response Analyzer     │      │
│                        │ produces cognitive    │      │
│                        │ snapshot              │      │
│                        └──────────┬────────────┘      │
│                                    │                   │
│              ┌─────────┬──────────┼──────────┐        │
│              ▼         ▼          ▼          ▼        │
│         Continue   Probe      Detour     Advance      │
│         (next      deeper     (pause,    (celebrate,  │
│          step)     (targeted  teach      move to      │
│              │      question) prereq,    next topic)  │
│              │         │      return)        │        │
│              │         │         │            │        │
│              └────┬────┘         │            │        │
│                   │              │            │        │
│                   ▼              │            ▼        │
│              Loop back ◄────────┘      Topic Selector │
│              to present                               │
│              next step                                │
│                                                       │
└─────────────────────────────────────────────────────┘
```

At every turn, the system is both **teaching** and **diagnosing**. Every response from the learner is simultaneously an opportunity to advance the lesson *and* to refine the system's understanding of the learner's cognition.

---

## 9. Persistent Memory and State

### What Persists

The system maintains a persistent state that survives across sessions:

**Learner Profile**
- Identity, background skills (self-reported and verified), learning style preference, session statistics

**Knowledge Map State**
- Every node's current status (the 8-state model)
- Verification history: which questions were asked, when, what the learner said, and whether it was correct, partial, incorrect, or confused
- Teaching history: when each node was taught, which representation was used, and the retest results

**Beliefs, Assumptions, Gaps, and Contradictions**
- All four categories of tracked cognition, per topic
- Each with its full history — when first detected, how it evolved, whether it was resolved

**Curriculum Position**
- Current topic, completed topics, skipped topics
- The dynamic topic order (which may differ from the original path as the system adapts)
- The detour stack (if the learner left mid-detour)

**Conversation History**
- Recent conversation turns (pruned to a configurable maximum, typically the last 100 turns)
- Each turn includes the topic, the type (tutor explains, learner questions, learner responds, detour start/end), and any cognitive snapshot attached to learner responses

**Global Statistics**
- Total questions asked, total detours triggered, how many detours actually helped, longest streak of topics understood without detours
- Per-lesson pass rates (percentage of learners who verify understanding without detours), tracked per lesson and per representation type — feeds the teaching effectiveness loop (see [Section 12a](#12a-teaching-effectiveness-loop))
- Misconception candidate log: novel misconceptions discovered from learner data, with occurrence counts and review status (see [Section 12b](#12b-misconception-discovery-propagation))
- Per-topic difficulty scores: composite measure of interrupt rate, time to verify, confusion density, and abandonment rate (see [Section 12d](#12d-aggregate-struggle-signals))

### Session Conversation Pruning and Summarization

Conversation history can't grow indefinitely. The system manages this through:

- **Rolling window**: Only the most recent turns are kept in full (typically 100). Older turns are pruned.
- **Summarization**: Before pruning old turns, the system extracts key insights — beliefs discovered, gaps found, misconceptions corrected — and preserves these in the topic knowledge records. The raw conversation text is discarded, but its cognitive content is retained.
- **Per-topic history**: Questions already asked for each topic are recorded separately from the conversation (so the system doesn't repeat questions even after conversation pruning).

### Cross-Session Continuity

The system maintains coherence across sessions spanning weeks or months through several mechanisms:

- **The learner model is the memory**: Rather than relying on conversation recall, the system encodes everything it learns about the learner into structured data. The Response Analyzer and Confusion Analyzer are responsible for this extraction — every interaction produces cognitive artifacts that persist independently of the conversation log.
- **Confidence decay creates natural re-engagement**: When a learner returns after a long absence, the system doesn't just pick up where it left off. Decayed nodes trigger re-verification, which serves double duty as review and as re-engagement ("Let's make sure you still remember X before we build on it").
- **Context re-establishment**: Each session opens with a brief summary of the learner's position and recent progress, generated from the persistent state rather than from replaying old conversations.

### Export and Import

The complete learner state can be exported as a portable file and imported on a different device or instance. This enables:

- **Device migration**: A learner who starts on one device can continue on another
- **Backup and recovery**: The learner's entire learning history is their data, not locked into a single application instance
- **Sharing**: A learner could share their profile with a human tutor to show exactly where they are in a domain

The export includes the full learner model, knowledge map state, and curriculum position — everything needed to resume exactly where the learner left off.

---

## 10. Interaction Design

### The Interrupt System

The learner can pause the lesson at **any point** through:
- A persistent "I don't understand" button (always visible)
- A keyboard shortcut (spacebar)
- Clicking directly on a term, code line, or diagram element they don't understand

When an interrupt occurs:
1. The current animation freezes in place
2. A soft overlay fades in, dimming the lesson content
3. The **question zone** slides up from the bottom

### The Question Zone

The question zone is the learner's voice. It provides:

- A free-text input for typing their question
- **Dynamic quick options** generated based on what's currently on screen, common confusion points for the current topic, and the learner's personal history of struggles
- Examples of quick options: "What does this term do?", "How does this connect to what we learned before?", "Can you show me a simpler example?", "I think I understand but I'm not sure"
- A submit button and a resume button (if they decide they actually are okay)

The quick options lower the barrier to asking for help. Many learners can't articulate *what* they don't understand — the quick options give them a starting point.

### The Detour Flow

After the learner asks their question:

1. The question zone shows a brief "thinking" state while the Confusion Analyzer runs
2. The system identifies the missing link
3. The question zone transforms into the **detour panel**, which explains: "I think the missing piece is understanding [concept]. Let me show you."
4. The detour mini-lesson plays within the detour panel (same animation system, shorter format)
5. At the end, a quick verification question
6. If the learner passes: celebration animation, the detour panel shrinks away, and the original lesson resumes
7. If still confused: the system tries a different angle or goes deeper (nested detour, max depth 3)

### Resume with Re-Contextualization

When returning from a detour:

1. The detour panel slides away
2. The overlay lifts
3. A brief "rewind" effect replays the last step before the interruption
4. A small breadcrumb appears: "← We detoured to learn about [concept]"
5. The lesson continues from the checkpoint

This re-contextualization is critical — after a detour that might have taken several minutes, the learner needs to be reminded of where they were and what they were thinking about before the interruption.

### Knowledge Map Visualization

The knowledge map visualization is the learner's map of their own understanding. It uses a **radial mind-map layout** where knowledge expands outward from the center:

**Visual Language**:
- **Core nodes**: Large, at the center. The major topics they represent.
- **Concept nodes**: Medium, radiating outward from their core. Sub-concepts nest within their parent concepts.
- **Detail nodes**: Small, at the periphery around their concept.
- **Foundation nodes**: Distinctly shaped (e.g., hexagonal), with connections spanning across multiple regions of the map — visually communicating their cross-cutting nature.
- **Dependencies**: Directed arrows showing prerequisite relationships. Hard dependencies are solid lines; helpful dependencies are dashed; optional dependencies are dotted.
- **Glow**: Known nodes emit a warm glow. The more verified, the brighter.
- **Pulse**: Gaps pulse gently — attracting attention without causing anxiety.
- **Cracks**: Misconceptions have a subtle fracture pattern — visually distinct from gaps, signaling something different is wrong.
- **Expansion animation**: When a concept is probed, its details expand outward from it with a radial animation — knowledge literally growing outward.
- **Connection sparks**: When a detail is learned and it connects to another part of the map, a spark travels along the dependency to show the connection forming. Foundation connections light up across distant regions of the map.

**Color Coding**:

| Color | Meaning |
|-------|---------|
| Green (bright) | Verified known — passed application-level test |
| Green (fading) | Previously verified, confidence decaying |
| Yellow | Assumed known — self-reported, unverified |
| Red | Identified gap — confirmed they don't know this |
| Orange | Misconception — confidently wrong (highest priority) |
| Purple | Currently being taught |
| Gray | Unknown — not yet assessed |

**Interaction**:
- Click any node to see its status, verification history, and dependencies
- Hover for a tooltip: "Electron carriers — verified through application test on Oct 15"
- Zoom into a region by clicking a core; zoom out to see the whole map
- A "journey trail" shows the path the learner has taken — which nodes they visited in what order

**Progressive Growth**:
At the start, only cores are visible at the center — major topics, dimly connected. As the learner is assessed, concepts radiate outward from the cores. As concepts are explored, details appear at the periphery. Foundations emerge with connections spanning across regions when the system identifies cross-cutting prerequisites. The map fills outward over time like expanding knowledge — dark areas illuminate, isolated clusters connect via shared foundations. This visual growth is one of the system's most powerful motivation tools — the learner can literally see their knowledge expanding outward from the center.

### Progress Tracking and Celebration

**Progress indicators**:
- A progress bar showing topic completion (e.g., "Topic 5 of 14 — 36%")
- Session statistics: detours this session, beliefs confirmed, time spent
- The constellation view: completed topics as bright connected stars, in-progress topics pulsing warmly, locked topics as dim dots

**Celebration moments** — when the learner demonstrates genuine understanding:
- A gentle particle burst effect (gold particles, gravity-affected)
- The newly understood concept "locks in" — slides into the knowledge bank
- The progress bar advances with a smooth fill animation
- The knowledge map gains a new bright node
- A new connection line draws to related concepts

These celebrations are earned — they only trigger on verified understanding (application-level or synthesis-level test), never on mere recall. This makes them meaningful: the learner knows that when the system celebrates, they genuinely got it.

---

## 11. Reference: What the System Tracks

This section provides plain-English descriptions of the key data concepts the system uses internally. These are conceptual descriptions, not code.

### Knowledge Node

A knowledge node is the fundamental unit of the knowledge map.

| Field | Description |
|-------|-------------|
| **ID** | Unique identifier for this node |
| **Label** | Human-readable name (e.g., "Persistent connections", "The Maillard reaction") |
| **Type** | Core, concept, detail, or foundation |
| **Parent** | The node this one belongs to (null for top-level cores) |
| **Children** | Nodes that belong to this one |
| **Status** | One of the 8 states: assumed_known, verified_known, shaky, misconception, identified_gap, being_taught, taught_untested, verified_learned |
| **Self-reported level** | What the learner claimed (none / beginner / intermediate / advanced), if applicable |
| **Verification history** | A list of every time this node was tested: the question asked, the learner's response, the result (correct / partial / incorrect / confused), and the timestamp |
| **Teaching record** | When this node was taught, which representation was used (diagram, flow trace, comparison, analogy, etc.), and the results of subsequent retests |
| **Blocked by** | Other nodes that must be green before this one can be taught |
| **Blocks** | Other nodes that depend on this one |
| **Dependency weight** | How strong the relationship to its parent is: hard, helpful, or optional |

### Learner Profile

The learner profile captures who the learner is and how they engage.

| Field | Description |
|-------|-------------|
| **ID** | Unique identifier, generated on first session |
| **Name** | The learner's name |
| **Created at** | When they first started |
| **Last session** | When they last engaged |
| **Total sessions** | How many sessions they've had |
| **Total time** | Cumulative time spent learning |
| **Background skills** | A map of prerequisite areas to skill levels. The specific areas depend on the domain (e.g., for a chemistry domain: math comfort, lab experience, prior chemistry courses; for a music domain: instrument experience, theory background, ear training) |
| **Learning style** | Preferred approach: concepts first, examples first, challenge-driven, or mixed |

### Topic Knowledge

For each topic in the domain, the system tracks this learner's engagement with it.

| Field | Description |
|-------|-------------|
| **Topic ID** | Which topic this record is for |
| **Status** | Locked, not started, in progress, understood, or mastered |
| **Time spent** | How long the learner has spent on this topic |
| **Beliefs** | Things the learner has explicitly stated they believe about this topic. Each tracks the claim, accuracy, confidence, and whether it's still active, has been revised, or was corrected. |
| **Assumptions** | Things the learner believes implicitly. Each tracks the assumption itself, the evidence that revealed it, and whether the system has surfaced and addressed it. |
| **Gaps** | Concepts the learner is missing. Each tracks the concept, why it matters, importance level, resolution status, and which other topics are affected by this gap. |
| **Contradictions** | Pairs of conflicting statements the learner has made, with the nature of the conflict and whether it's been resolved. |
| **Questions already asked** | Prevents the system from repeating questions |
| **Detour history** | Every detour triggered during this topic: what caused it, what gap it addressed, the mini-lesson topic, and whether it helped |
| **Interrupt count** | How many times the learner has paused with questions on this topic (a useful signal for topic difficulty relative to this learner). Detour history across all learners is aggregated for pattern analysis — see [Section 12c](#12c-detour-pattern-analysis) for how the system uses detour frequency and success rates to improve prerequisite chains. |

### Curriculum State

The curriculum state tracks where the learner is in their learning journey.

| Field | Description |
|-------|-------------|
| **Current topic** | What they're studying right now (null if between topics) |
| **Completed topics** | Topics they've finished, in the order they were completed |
| **Skipped topics** | Topics that were skipped (because their background already covers them) |
| **Topic order** | The dynamic, personalized sequence — this may differ from the default order because the system reorders based on the learner's needs |
| **Detour stack** | If the learner is currently on a detour (or nested detour), this stack records: which topic they were on, where in the lesson they were, and what question triggered the detour. When the detour resolves, the stack pops and the system returns to the previous context. |

### Cognitive Snapshot

A cognitive snapshot is the analysis output produced every time the learner responds to a question or asks one. It is the primary mechanism by which the system updates its understanding of the learner.

| Field | Description |
|-------|-------------|
| **New beliefs** | Claims the learner just made, with accuracy assessment and confidence level |
| **New assumptions** | Implicit beliefs detected in what they said, with the evidence |
| **Gaps revealed** | Concepts the learner is missing that this response exposed, with importance |
| **Gaps resolved** | Previously identified gaps that this response addresses |
| **Contradictions** | Conflicts between what they just said and what they've said before |
| **Ready to advance** | Whether the learner is ready to move forward |
| **Missing link** | If they're stuck: the specific prerequisite concept they lack, why it blocks them, and a suggested mini-lesson to address it |
| **Suggested next action** | The system's recommendation: continue lesson, probe deeper, start detour, try a challenge, revisit an earlier topic, or celebrate and advance |
| **Best next question** | The single most informative question to ask next |

### Conversation Turn

A single turn in the ongoing dialogue between the system and the learner.

| Field | Description |
|-------|-------------|
| **Timestamp** | When this turn occurred |
| **Topic** | Which topic was being discussed |
| **Type** | The nature of this turn: tutor explains, learner questions, learner responds, tutor follow-up, detour start, or detour end |
| **Content** | The actual text of what was said |
| **Cognitive snapshot** | For learner responses only — the analysis of what this response revealed |

---

## 12. System Self-Improvement — Feedback Loops

The system does not just teach — it learns from its own teaching. Four feedback loops continuously improve the quality of the learning experience by analyzing aggregate patterns across all learners.

### 12a. Teaching Effectiveness Loop

Every lesson generates effectiveness data that the system tracks and acts on.

**Metrics tracked per lesson**:
- Pass rate: percentage of learners who verify understanding without requiring detours
- Detour rate: percentage of learners who trigger at least one detour
- Average attempts to verify: mean number of questions before a node reaches `verified_learned`
- Representation used: which teaching format was selected (diagram, flow trace, side-by-side, etc.)
- Learner archetype: the learner's learning style (concepts first, examples first, challenges, mixed)

**Decision rules**:

| Condition | Action |
|-----------|--------|
| Pass rate < 65% over 30+ learners | Flag lesson for review — the lesson itself may be poorly structured, not the learners |
| One representation's pass rate is >15 percentage points higher than another for the same concept | Prefer the higher-performing representation for future learners |
| Pass rate varies >20 percentage points by learning style | Flag lesson for style bias — the lesson may over-serve one learning style at the expense of others |

**Representation effectiveness matrix**: Over time, the system builds a matrix mapping `(knowledge type × learner archetype) → best representation`. For example, the matrix might show that "process/sequence" knowledge is best taught via flow traces for "concepts first" learners but via worked examples for "examples first" learners. This matrix starts empty and is populated as data accumulates — no representation preference is hard-coded.

### 12b. Misconception Discovery Propagation

The initial misconception catalog comes from domain experts and source material, but learners reveal novel misconceptions that no expert anticipated.

**Discovery pipeline**:

1. **Record**: When the Response Analyzer detects a confident incorrect belief that doesn't match any known misconception in the catalog, it records a **misconception candidate**: the concept it relates to, the exact learner statement, the evidence (question asked, context), and a timestamp.

2. **Accumulate**: Each candidate is matched against existing candidates by semantic similarity. If a new candidate matches an existing one, the occurrence count increments.

3. **Promote at 3 occurrences**: When 3 independent learners (different learner IDs) produce the same misconception candidate, it is promoted to **"emerging misconception"** status:
   - Added to the misconception catalog with status `unreviewed`
   - The system begins designing assessment questions that specifically test for this misconception
   - Domain experts are alerted for review

4. **Expert review**:
   - Expert confirms → misconception is permanently added to the catalog with status `confirmed`, assessment questions are finalized
   - Expert rejects → the candidate is marked `rejected`, and the rejection reason is fed back to improve the Response Analyzer's misconception detection (reducing false positives of that type)

5. **Escalation at 10 occurrences**: If a misconception candidate reaches 10 independent occurrences without expert review, it is escalated with a **48-hour auto-inclusion deadline**. If no expert responds within 48 hours, the misconception is automatically included in the catalog with status `auto-included` — because at 10 independent occurrences, the cost of ignoring it exceeds the risk of a false positive.

### 12c. Detour Pattern Analysis

Detours are the system's mechanism for handling missing prerequisites discovered during teaching. Analyzing detour patterns across learners reveals structural problems in the knowledge map.

**Metrics tracked per (topic, detour_target) pair**:
- Frequency: what percentage of learners studying this topic need a detour to this target
- Success rate: what percentage of those detours result in the learner subsequently passing the original topic
- Depth: how deeply the detour nests (1 = direct prerequisite, 2 = prerequisite of prerequisite, etc.)

**Decision rules**:

| Condition | Action |
|-----------|--------|
| >30% of learners need the same detour | Recommend making it a **proactive prerequisite check** — ask about this prerequisite before starting the topic, rather than discovering the gap mid-lesson |
| >50% of learners need the same detour | **Auto-promote** to mandatory prerequisite check — the system automatically verifies this prerequisite before teaching the topic |
| Detour success rate < 50% | Flag that the **real gap is deeper** — the detour is teaching the wrong thing, or the prerequisite itself has prerequisites that aren't being addressed |
| Average detour depth > 2.0 for a topic | Flag **incomplete prerequisite chain** — the knowledge map is missing intermediate dependencies between this topic and its foundations |

### 12d. Aggregate Struggle Signals

The system monitors per-topic metrics that reveal which topics are structurally difficult — not just hard for one learner, but consistently challenging across the population.

**Metrics tracked per topic**:
- Interrupt rate: average number of "I don't understand" interrupts per learner
- Time to verify: average time from first lesson step to `verified_learned`
- Confusion density: number of Confusion Analyzer invocations per lesson
- Abandonment rate: percentage of learners who start the topic but leave it in `in_progress` status for >14 days

**Difficulty score**:

```
difficulty_score = (interrupt_rate / global_avg_interrupt)
                 × (time_to_verify / global_avg_time)
                 × (confusion_density / global_avg_confusion)
                 × (1 + abandonment_rate)
```

A score of 1.0 means the topic is average difficulty. A score of 3.0 means it's 3× harder than average across all measured dimensions.

**Decision rules**:

| Condition | Action |
|-----------|--------|
| Difficulty score > 2.5 | Flag for **comprehensive review** — examine lesson quality, prerequisite completeness, and misconception coverage |
| Abandonment rate > 20% | Alert about **motivation/difficulty cliff** — this topic may need to be broken into smaller sub-topics or preceded by a gentler introduction |

**Redesign priority list**: Topics flagged by any of the above rules are ranked by difficulty score into a redesign priority list. This list is the system's answer to "where should we invest effort to improve the learning experience?" — it surfaces the topics where structural improvements would help the most learners.

---

## 13. Error Recovery and Graceful Degradation

**Governing principle**: Never leave the learner stranded. Every failure has a fallback. Every fallback has an escalation.

The system operates in an inherently uncertain environment — learners give ambiguous answers, AI agents produce imperfect output, and diagnostic algorithms sometimes can't converge. This section defines what happens when things go wrong.

### 13a. Diagnostic Failure

**Trigger**: The diagnostic narrowing algorithm (Section 5) exhausts its 4-question budget without achieving >0.8 confidence in the gap location.

**Recovery sequence**:

1. **Present top hypotheses to the learner**: "I've been trying to figure out exactly where the confusion is, and I have a few ideas. Which of these feels closest to where you're stuck?"
   - Display the top 3 hypotheses as plain-language options (e.g., "I'm not sure what electron carriers are," "I understand the pieces but not how they connect," "I think I might have the wrong idea about how energy is transferred")
   - Include a "None of these" option

2. **If learner selects a hypothesis**: Accept it as the gap location (confidence override), mark the node as `identified_gap`, and begin teaching. The system notes that this gap was learner-identified rather than algorithmically confirmed.

3. **If learner selects "None of these"**: The gap is deeper or more unusual than the system's model predicts.
   - Run a **focused foundation assessment**: test the 3–5 most foundational prerequisites of the current topic with direct questions.
   - If foundations are weak (≥2 fail): teach the weakest foundation first. The original confusion was likely downstream of a foundational gap.
   - If foundations are solid (≤1 fails): the gap is in an unexpected location. Flag the topic for instructor review.

4. **If no instructor is available** (self-study mode): Offer the learner a choice:
   - "Let me try explaining this differently" → re-explain the current concept using a different representation
   - "Let me skip this for now and come back later" → bookmark the topic, move to the next one, and schedule a return visit
   - "Let me review the prerequisites" → start a quick review of the topic's hard dependencies

### 13b. Detour Depth Exceeded

**Trigger**: Detour nesting reaches the maximum depth of 3 — the system has gone three levels deep into prerequisites and still hasn't reached solid ground.

**Recovery sequence**:

1. **Pop the entire detour stack**: Abandon all nested detours. The learner is returned to the context of the original topic (not a nested prerequisite). A brief message explains: "We've gone pretty deep into prerequisites. Let me take a step back and try a different approach."

2. **Run focused foundation assessment**: Test the 5 most foundational prerequisites of the original topic (not the detour chain) with direct questions.

3. **Branch based on results**:
   - If < 3 foundation gaps found: Teach the missing foundations directly (not as detours — as explicit lessons). Once foundations are solid, resume the original topic.
   - If ≥ 3 foundation gaps found: Recommend a **prerequisite learning path** — "Before we tackle [original topic], I think we need to build up your foundation in [area]. Here's what I recommend we cover first..." Generate a mini-learning-path for the prerequisite area.

4. **If learner insists on continuing** despite the recommendation: Allow it with a warning ("We can try, but you may find it harder without these foundations"). Increase the depth tolerance to 4 for this topic only, and flag the session for review. The system respects learner autonomy while making its recommendation clear.

### 13c. Agent Output Failure

**Trigger**: An AI agent (Lesson Generator, Confusion Analyzer, Response Analyzer, etc.) produces output that fails validation.

**Detection criteria**:
- **Structural validation**: Output doesn't match the expected schema (missing required fields, wrong types)
- **Coherence check**: Output references concepts not in the current domain, contradicts the knowledge map, or is unrelated to the current topic
- **Length bounds**: Output is <10% or >300% of expected length for that agent's typical output

**Recovery sequence**:

1. **Retry with modified prompt** (up to 2 retries):
   - First retry: Add explicit constraints to the prompt ("You are teaching [topic] to a learner who [context]. Your output must include [required fields].")
   - Second retry: Simplify the prompt, reduce the scope of what the agent is asked to produce.

2. **Fall back to template responses**: If retries fail, use pre-built template responses:
   - Lesson Generator → use a generic lesson template for the knowledge type (e.g., standard "definition + example + question" template for a terminology node)
   - Confusion Analyzer → ask the learner to rephrase their confusion, then route to the Quick Clarification Handler
   - Response Analyzer → treat the response as "partial understanding," ask a simpler follow-up question
   - Template responses are less personalized but guaranteed to be structurally correct and on-topic.

3. **If no template exists**: Ask the learner to narrow their question or rephrase what they're trying to learn. Use their response as a fresh starting point for the next agent invocation.

4. **Logging**: Every agent failure is logged with the input that caused it, the invalid output, the recovery path taken, and the eventual outcome. The system tracks failure rate per agent per topic — a high failure rate for a specific (agent, topic) pair signals that the topic's content specification needs improvement.

### 13d. Ambiguous Learner Response

**Trigger**: The Response Analyzer cannot classify a learner's response with confidence. The response is not clearly correct, incorrect, partial, or confused — it resists categorization.

**Recovery sequence**:

1. **Targeted follow-up question**: Ask a narrower question that constrains the possible interpretations. For example, if the learner's response to "How do electron carriers work?" is vague and meandering, follow up with: "Specifically: does NADH carry electrons to the electron transport chain, or does it carry something else?"

2. **If still ambiguous**: Reduce to a **binary yes/no question** that the system can interpret unambiguously. For example: "Is it true that NADH delivers electrons to the electron transport chain? Yes or no."

3. **If still ambiguous after 2 follow-up attempts**: The system cannot determine the learner's understanding of this node through automated means.
   - Mark the node with the `needs_human_review` overlay (see [Section 3, Node Status Model](#node-status-model))
   - Continue the lesson while avoiding dependence on that node — route around it in the teaching sequence
   - Schedule re-assessment of that node at the beginning of the next session, using a different question format
   - Log the ambiguous interaction for analysis (it may reveal a question design problem rather than a learner problem)

---

*This document synthesizes the complete design of the adaptive learning system. The system is domain-agnostic: the same diagnostic engine, teaching strategy, agent architecture, and learner model apply whether teaching molecular biology, software engineering, music theory, or culinary arts. The domain changes; the cognitive machinery does not. Knowledge is modeled as a radial map — cores at the center, concepts radiating outward, details at the periphery, foundations spanning across regions. Diagnosis narrows outward to find gaps; teaching repairs inward to fix them. The system improves its own teaching over time through four feedback loops that analyze teaching effectiveness, discover novel misconceptions, detect prerequisite gaps, and surface structurally difficult topics (Section 12). And when things go wrong — diagnostic failures, excessive detour depth, agent errors, or ambiguous responses — the system degrades gracefully, always ensuring the learner has a clear path forward (Section 13).*
