---
name: design-thinking
description: Use when making architectural or design decisions. Covers module depth, interface design, tradeoff analysis, and avoiding the convergence trap. Synthesizes from architecture improvement, design consultation, and critical thinking patterns.
---
# Design Thinking: Architecture, Decisions, and Tradeoffs

## Core Principles

### 1. Think Before Coding
Never start implementation until you understand the design space. Explicitly state assumptions. Discuss tradeoffs. Ask questions when requirements are unclear. Guessing is the most expensive form of engineering.

### 2. Simplicity First
Implement the minimal solution. Speculative features, unnecessary abstractions, and premature generalization are the root cause of most architectural debt. Every line of code you don't write is a line that can never have a bug.

### 3. Surgical Changes
Modify existing code as little as possible. Maintain existing style and conventions. Don't refactor unrelated code "while you're in there" — each change should do exactly one thing. Clean up only what your change directly impacts.

### 4. Depth Over Breadth
A module is "deep" when it has a simple interface and complex implementation — it does a lot with minimal surface area. A module is "shallow" when its interface is as complex as its implementation — it adds friction without leverage. Design deep modules.

---

## Phase 1: Understand the Territory

### Zoom out first
When you enter an unfamiliar area of the codebase, start at the highest level and work down:

1. **Project map**: What are the major packages/directories? How do they relate?
2. **Domain glossary**: What terms does the project use? What do they mean? (Check `AGENTS.md`, `CONTEXT.md`, or existing skill files.)
3. **Data flow**: How does data enter the system, get transformed, and exit?
4. **Existing ADRs / design docs**: What decisions were already made? Why?
5. **Change patterns**: `git log --all -- <directory>` — what have past contributors done here?

### Check architecture decision records
If ADRs exist at `docs/adr/` or similar, read the relevant ones before proposing any change. Past decisions carry rationale you need to understand.

---

## Phase 2: Frame the Decision

### State the problem clearly
Before exploring solutions, articulate:
- **What specific problem are we solving?**
- **What constraints exist?** (Time, budget, compatibility, skill set)
- **What is NOT changing?** (Scope boundaries are as important as scope)
- **How will we know we succeeded?** (Verifiable criteria)

### Generate options
Produce 2-4 distinct approaches. Avoid the temptation to converge on one idea too early. Each option should be a real alternative, not a straw man.

For each option, state:
- **The approach**: What it does at a high level
- **The interface**: How other modules interact with it
- **Tradeoffs**: What it gains and what it sacrifices
- **Risk areas**: Where it could go wrong### Evaluate systematically

Use this framework to evaluate each option:

| Criterion | Question |
|---|---|
| **Simplicity** | Does this minimize overall complexity, not just move it around? |
| **Locality** | Will a change require touching many files or just a few? |
| **Testability** | Can behavior be verified through public interfaces? |
| **Evolution** | Does this allow for future changes without rewriting? |
| **Cognitive load** | How much does a developer need to know to use this? |

### Evidence Evaluation

Every design option rests on claims — about the problem, the technology, the tradeoffs. Not all claims are equal. Evaluate the evidence behind each claim before deciding.

| Evidence level | What it means | Engineering example |
|---|---|---|
| **Measured** | Direct observation from your system or environment | "We have APM traces showing 500ms p99 latency on this endpoint" |
| **Documented** | Recorded in authoritative docs or specs | "The Postgres docs state that index-only scans skip heap lookups" |
| **Demonstrated** | Shown via a reproduction, prototype, or proof of concept | "I wrote a script that proves this caching strategy reduces DB calls by 40% on our data" |
| **Experienced** | Based on personal or team experience with similar situations | "We tried microservices before and the operational cost was too high" |
| **Inferred** | Reasonable logical extrapolation from known facts | "Since this grows O(n²), it will be slow with 10K users" |
| **Assumed** | Taken as true without direct evidence | "This library must handle authentication" — check the docs |

**Rule of thumb**: Base decisions on the strongest available evidence level. If the critical claim for an option relies on "assumed" or "inferred" evidence, downgrade that option's confidence. Flag those assumptions for explicit verification before committing to the design.

**When evidence conflicts**:
- Measured > Documented > Demonstrated > Experienced > Inferred > Assumed
- If two sources conflict at the same level, investigate why before deciding
- Be skeptical of evidence that conveniently supports your preferred option (confirmation bias — see below)

---

## Phase 3: Interface Design

### The interface is the foundation
When designing a module, the interface (public API) matters more than the implementation. A bad interface locked in early will create pain across the entire codebase.

### Interface design checklist:
- **Minimal surface area**: Can any part of this interface be simplified or removed?
- **Consistent abstractions**: Does it use the same patterns as the rest of the codebase?
- **Domain language**: Does it use the project's vocabulary?
- **Hard to misuse**: Can the user get it wrong? If so, redesign.
- **Testable through the interface**: Can you verify behavior without mocking internals?

### Seams and adapters
A "seam" is a place where you can alter behavior without changing the code that uses it. Identify the seams in your design:
- Dependency injection points
- Adapter interfaces (e.g., database, HTTP client, file system)
- Configuration boundaries
- Callback/hook points

---

## Phase 4: Tradeoff Analysis

### Common engineering tradeoffs

| Tradeoff | Default bias | When to go the other way |
|---|---|---|
| Simplicity vs Flexibility | Bias toward simplicity | When you have 3+ known future use cases |
| Speed vs Correctness | Bias toward correctness (for core logic) | For prototypes and explorations |
| Consistency vs Innovation | Bias toward consistency | When existing patterns actively fight your use case |
| Coupling vs Duplication | Bias toward tolerated duplication | When the coupling pattern is well-understood (DRY only when stable) |
| Build vs Buy | Bias toward build (for core domain) | For commodity infrastructure |
| Optimize now vs Later | Bias toward later | When you can measure a real bottleneck |

### When to avoid design work:
- The code is a prototype or will be thrown away
- The requirements are fundamentally unclear
- There is no evidence the current design is causing problems
- The change is cosmetic and affects nothing measurable

---

## Phase 5: Critique Loop

Before finalizing any design, critique it systematically:

### Self-critique:
1. **What am I assuming that might be wrong?** State each assumption explicitly.
2. **What's the simplest version of this?** Remove features until it hurts, then add back only what's essential.
3. **Where does this break?** Trace error paths, edge cases, and failure modes.
4. **Who else touches this?** Think about the other modules and developers this affects.

### Peer review mindset:
When reviewing code or designs, follow these principles:
- **Evaluate technically, not socially.** The design isn't yours or theirs — it's the codebase's.
- **Verify before accepting.** Treat all suggestions as hypotheses to be validated against codebase reality.
- **Push back when necessary.** If a suggestion is technically incorrect, lacks context, or introduces unnecessary complexity, say so with technical reasoning.
- **Be specific.** "This approach doesn't handle X" is useful. "I don't like this" is not.

### Bias Detection Checklist

Before settling on a decision, check for these common engineering biases:

| Bias | Definition | How to counteract |
|---|---|---|
| **Confirmation bias** | Seeking evidence that supports your preferred option | Actively argue for the alternatives. Assign someone to play devil's advocate. |
| **Recency bias** | Preferring the approach you just used or learned about | Ask: "If I had never used this technology, would I still pick it?" |
| **Anchoring** | Being disproportionately influenced by the first option considered | Generate options without evaluating them. Evaluate all of them before comparing. |
| **Status quo bias** | Preferring the current approach because it's familiar | Evaluate the current approach with the same rigor as alternatives. Does it still fit? |
| **Not Invented Here** | Rejecting existing solutions to build your own | Check what already exists in the codebase, ecosystem, or team. |
| **Sunk cost** | Continuing with a bad approach because time was invested | The best time to change course is now. The time already spent is gone. |
| **Premature convergence** | Settling on the first viable solution without exploring alternatives | Mandate at least 3 options before any discussion of tradeoffs. |
| **Overconfidence** | Underestimating the complexity or risk of your preferred approach | Estimate effort for your approach, then double it. Estimate alternatives the same way. |

### Assumption Audit

Every design rests on assumptions. Some are safe, some are lethal. Before finalizing, audit them:

1. **List every assumption your design makes.** Be exhaustive:
   - About the user (behavior, volume, patterns)
   - About the infrastructure (latency, availability, scale)
   - About the data (shape, size, growth rate)
   - About dependencies (stability, compatibility, security)
   - About the team (familiarity, capacity, expertise)

2. **Classify each assumption:**
   - 🟢 **Safe**: Almost certainly true (e.g., "the database will return rows as objects")
   - 🟡 **Reasonable**: Likely true but should be validated soon (e.g., "we won't exceed 100 concurrent users in year one")
   - 🔴 **Risky**: May be false and would break the design if it is (e.g., "this third-party API has 99.99% uptime")

3. **For each 🔴 assumption**, specify:
   - How to validate it (measurement, experiment, research)
   - What the fallback is if it proves false
   - At what point it becomes a blocker (deadline, scale, event)

---

## Quick Reference: Design Decision Workflow

```
1. Understand territory ─► 2. Frame problem ─► 3. Generate options
                                                  │
                           ┌───────────────────────┘
                           ▼
                   4. Evaluate systematically
                           │
                           ▼
                   5. Design interface ─► 6. Critique ─► 7. Decide
                           │
                           ▼
                   8. Write minimal implementation
                           │
                           ▼
                   9. Verify against criteria
```

### When to stop designing and start coding:
- You have a clear, minimal interface
- You can articulate the tradeoffs you're making
- You know what "done" looks like
- You have a plan to validate (test, prototype, or review)

---

## Common Failure Modes

| Mode | Symptom | Antidote |
|---|---|---|
| **Over-engineering** | Designing for futures that never arrive | Ask: "What's the simplest thing that works?" |
| **Analysis paralysis** | Endless tradeoff discussion with no decision | Pick the option with the best exit strategy |
| **Convergence trap** | Everyone agreeing on the first plausible idea | Force yourself to generate at least 3 options |
| **Premature optimization** | Complicating code for performance gains not yet measured | Measure first, optimize second |
| **Sunk cost** | Continuing with a bad design because time was invested | The best time to change course is now |
| **Not invented here** | Rejecting existing solutions to build your own | Check what the project already does before designing |
| **Evidence blindness** | Basing decisions on assumed rather than measured evidence | Classify each claim's evidence level. Downgrade assumed claims. |
| **Anchoring on first option** | The first plausible approach distorts evaluation of all alternatives | Generate options blind. Evaluate them blind. Only then reveal which is which. |
| **Overconfidence in estimates** | Assuming timelines and complexity are predictable | Double estimates. Add uncertainty ranges. Plan for the worst case.
