---
name: plan
description: Create a decision-complete implementation plan before writing code, covering scope, steps, risks, and test strategy.
---
# Plan

Create a structured implementation plan before writing code. The goal is not just to describe the work, but to remove ambiguity so implementation can start with clear decisions already made.

## When to Use

- Starting a new feature or subsystem
- Making architectural or cross-file changes
- Refactoring work with dependencies or risk
- Any request where the next step is better planning than immediate implementation

## Working Rules

1. Analyze the existing codebase first.
2. Identify dependencies, constraints, and relevant patterns.
3. Resolve discoverable facts by inspection before asking questions.
4. Ask only for preferences, tradeoffs, or missing intent that cannot be inferred.
5. Keep the plan decision-complete: every step should be specific enough to execute without extra judgment.
6. Include testing, validation, and any open risks or assumptions.

## What Good Plans Include

- Clear objective and scope
- In-scope and out-of-scope boundaries
- Exact files, modules, or components to touch
- The pattern to follow from existing code when relevant
- Implementation order with verifiable steps
- Error handling, edge cases, and test strategy
- Risks, assumptions, and unresolved questions

## Output Format

```markdown
# [Project / Task Name] Plan

## Objective
[One-sentence summary of what we're building]

## Scope

### In scope
- [Specific item]
- [Specific item]

### Out of scope
- [Explicitly excluded item]
- [Explicitly excluded item]

## Steps

1. **[Title]**
   - **Details**: [Inspect relevant files, APIs, and existing patterns.]
   - **Verification**: [What proves this step is complete]

2. **[Title]**
   - **Details**: [Choose the approach and note the files or modules to change.]
   - **Verification**: [What should be true after the design step]

3. **[Title]**
   - **Details**: [List exact edits in execution order.]
   - **Verification**: [How to validate the implementation]

4. **[Title]**
   - **Details**: [Run the relevant checks and confirm expected behavior.]
   - **Verification**: [Tests, checks, or manual verification to perform]

## Risks & Tradeoffs
- [Main risk or tradeoff]
- [Main risk or tradeoff]

## Open Questions
- [Only questions that cannot be answered by inspection]
- [Only questions that cannot be answered by inspection]

## Test Strategy
- [How the feature will be tested]
- [Edge cases to cover]
```

## Quality Bar

A strong plan should leave the implementer with no major decisions to make. If a detail can be discovered, discover it. If a choice depends on intent, surface the choice clearly.
