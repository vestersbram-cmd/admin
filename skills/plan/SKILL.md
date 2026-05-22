---
name: plan
description: Create a decision-complete implementation plan before writing code. Covers design thinking (understand territory, frame decisions, interface design, tradeoff analysis, critique) and structured plan output.
---

# Plan

Create a structured implementation plan before writing code. The goal is not just to describe the work, but to remove ambiguity so implementation can start with clear decisions already made.

## Core Principles

### 1. Think Before Coding
Never start implementation until you understand the design space. Explicitly state assumptions. Discuss tradeoffs. Ask questions when requirements are unclear. Guessing is the most expensive form of engineering.

### 2. Simplicity First
Implement the minimal solution. Speculative features, unnecessary abstractions, and premature generalization are the root cause of most architectural debt. Every line of code you don't write is a line that can never have a bug.

### 3. Scope Each Change Precisely
A plan should define exactly what changes and what stays untouched. Each step should modify exactly one logical unit. Resist plans that bundle refactoring, cleanup, and feature work into one step — each change does exactly one thing. What you leave alone is as important as what you touch.

---

## When to Use

- Starting a new feature or subsystem
- Making architectural or cross-file changes
- Refactoring work with dependencies or risk
- Any request where the next step is better planning than immediate implementation
- Facing an ambiguous design decision that needs structured evaluation

### When NOT to Plan

Skip formal planning when:
- The code is a prototype or will be thrown away
- The requirements are fundamentally unclear (planning can't fix unknown unknowns)
- There is no evidence the current design is causing problems
- The change is cosmetic and affects nothing measurable

---

## Phase 1: Understand the Territory

Before proposing any solution, start at the highest level and work down.

### Zoom out first

1. **Project map**: What are the major packages/directories? How do they relate?
2. **Domain glossary**: What terms does the project use? What do they mean? (Check `AGENTS.md`, `CONTEXT.md`, or existing skill files.)
3. **Data flow**: How does data enter the system, get transformed, and exit?
4. **Existing ADRs / design docs**: What decisions were already made? Why? (Check `docs/adr/` or similar.)
5. **Change patterns**: `git log --all -- <directory>` — what have past contributors done here?
6. **Codebase conventions**: What patterns, lint rules, testing habits, and import styles exist in neighboring code?

### Checklist before planning:
- [ ] I understand the relevant parts of the codebase
- [ ] I know the project's conventions and patterns
- [ ] I can articulate the problem in one sentence
- [ ] I know what "done" looks like
- [ ] I have read existing ADRs or design docs for context

---

## Phase 2: Frame the Decision

Before exploring solutions, articulate the decision clearly.

### State the problem

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
- **Risk areas**: Where it could go wrong

### Evaluate systematically

Use this framework to evaluate each option:

| Criterion | Question |
|---|---|
| **Simplicity** | Does this minimize overall complexity, not just move it around? |
| **Locality** | Will a change require touching many files or just a few? |
| **Testability** | Can behavior be verified through public interfaces? |
| **Evolution** | Does this allow for future changes without rewriting? |
| **Cognitive load** | How much does a developer need to know to use this? |

### Evidence Evaluation

Every design option rests on claims — about the problem, the technology, the tradeoffs. Evaluate the evidence behind each claim before deciding.

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
- Be skeptical of evidence that conveniently supports your preferred option (confirmation bias)

---

## Phase 3: Design the Interface

### The interface is the foundation

When designing a module, the interface (public API) matters more than the implementation. A bad interface locked in early will create pain across the entire codebase.

### Interface design checklist:
- **Minimal surface area**: Can any part of this interface be simplified or removed?
- **Consistent abstractions**: Does it use the same patterns as the rest of the codebase?
- **Domain language**: Does it use the project's vocabulary?
- **Hard to misuse**: Can the user get it wrong? If so, redesign.
- **Deep interface**: Simple API surface with real work happening behind it (not a shallow pass-through)
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

---

## Phase 5: Write the Plan

After thinking through the design, produce a structured implementation plan.

### Working Rules

1. Analyze the existing codebase first.
2. Identify dependencies, constraints, and relevant patterns.
3. Resolve discoverable facts by inspection before asking questions.
4. Ask only for preferences, tradeoffs, or missing intent that cannot be inferred.
5. Keep the plan decision-complete: every step should be specific enough to execute without extra judgment.
6. Include testing, validation, and any open risks or assumptions.

### What Good Plans Include

- Clear objective and scope
- In-scope and out-of-scope boundaries
- Exact files, modules, or components to touch
- The pattern to follow from existing code when relevant
- Implementation order with verifiable steps
- Error handling, edge cases, and test strategy
- Risks, assumptions, and unresolved questions

### Plan granularity:
- **Too coarse**: "Implement the feature" — not actionable
- **Too fine**: "Import React, define props interface, write render function..." — too much overhead
- **Just right**: Steps that map to verification points (one test, one function, one file)

### Output Format

```json
{
  "summary": "Brief description of the plan",
  "plan": {
    "objective": "What we're building",
    "scope": {
      "in": ["Included work"],
      "out": ["Explicitly excluded work"]
    },
    "steps": [
      {
        "title": "Research and confirm constraints",
        "details": "Inspect relevant files, APIs, and existing patterns.",
        "verification": "What proves this step is complete"
      },
      {
        "title": "Design the solution",
        "details": "Choose the approach and note the files or modules to change.",
        "verification": "What should be true after the design step"
      },
      {
        "title": "Implement the change",
        "details": "List exact edits in execution order.",
        "verification": "How to validate the implementation"
      },
      {
        "title": "Test and verify",
        "details": "Run the relevant checks and confirm expected behavior.",
        "verification": "Tests, checks, or manual verification to perform"
      }
    ],
    "risks": ["Main risks or tradeoffs"],
    "open_questions": ["Only questions that cannot be answered by inspection"]
  }
}
```

---

## Phase 6: Critique Loop

Before finalizing any design, critique it systematically.

### Self-critique:
1. **What am I assuming that might be wrong?** State each assumption explicitly.
2. **What's the simplest version of this?** Remove features until it hurts, then add back only what's essential.
3. **Where does this break?** Trace error paths, edge cases, and failure modes.
4. **Who else touches this?** Think about the other modules and developers this affects.

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

### Peer review mindset:
- **Evaluate technically, not socially.** The design isn't yours or theirs — it's the codebase's.
- **Verify before accepting.** Treat all suggestions as hypotheses to be validated against codebase reality.
- **Push back when necessary.** If a suggestion is technically incorrect, lacks context, or introduces unnecessary complexity, say so with technical reasoning.
- **Be specific.** "This approach doesn't handle X" is useful. "I don't like this" is not.

---

## Quality Bar

A strong plan should leave the implementer with no major decisions to make. If a detail can be discovered, discover it. If a choice depends on intent, surface the choice clearly.

After the plan is written, verify against these gates:

- [ ] The objective is stated clearly in one sentence
- [ ] Scope boundaries (in/out) are explicit
- [ ] Each step is verifiable — you'll know when it's done
- [ ] Risks are acknowledged, not hidden
- [ ] Assumptions are surfaced, not assumed
- [ ] The plan is decision-complete — no ambiguity left for the implementer

---

## Common Failure Modes

| Mode | Symptom | Antidote |
|---|---|---|
| **Jumping in** | Writing code before understanding the codebase | Phase 1 is mandatory. Don't skip. |
| **Analysis paralysis** | Endless planning without writing code | Set a timer. 10 minutes max for planning simple tasks. |
| **Over-engineering** | Designing for futures that never arrive | Ask: "What's the simplest thing that works?" |
| **Convergence trap** | Settling on the first viable solution | Force yourself to generate at least 3 options. |
| **Premature optimization** | Complicating code for theoretical performance | Measure first. If you can't measure, don't optimize. |
| **Sunk cost** | Continuing a bad approach because time was invested | The best time to change course is now. |
| **Skipping evidence** | Basing decisions on assumptions, not facts | Classify each claim's evidence level. |
| **Scope creep** | Fixing everything you encounter | Each task has a scope. Write it down. Don't stray. |

---

## Quick Reference

```
┌──────────────────────────────────────────────────────────────────┐
│                    PLAN WORKFLOW                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Phase 1: Understand Territory ──► Phase 2: Frame Decision       │
│      │                                     │                      │
│      └── Project map, ADRs, conventions    │                      │
│      └── Domain glossary, data flow        │                      │
│                                            │                      │
│                      ┌──────────────────┘                        │
│                      ▼                                            │
│           Phase 3: Design Interface                                │
│                      │                                            │
│                      ▼                                            │
│           Phase 4: Analyze Tradeoffs                               │
│                      │                                            │
│                      ▼                                            │
│           Phase 5: Write the Plan (JSON)                          │
│                      │                                            │
│                      ▼                                            │
│           Phase 6: Critique ──► Quality Bar                       │
│               │                                                   │
│               └── Bias check, assumption audit                    │
│               └── Self-critique, peer review                      │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```
