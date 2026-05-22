---
name: workflow
description: Use when starting a new development task, planning implementation, or organizing a multi-step project. Covers the full lifecycle: understanding requirements, planning, executing, reviewing, and capturing lessons.
---

# Workflow: Structured Development Lifecycle

## Philosophy

Good AI-assisted development is not about doing everything automatically. It's about being methodical:

- **Understand before acting.** The most expensive code is code you have to rewrite.
- **Plan before executing.** A good plan survives contact with reality. A bad plan doesn't survive first contact with the codebase.
- **Verify before claiming.** Evidence is the only truth.
- **Learn from everything.** Every session produces knowledge. Capture it.

---

## Phase 1: Context Gathering

Before writing any code, gather broad context:

1. **Explore the territory**
   - What does this project do? (README, package.json, top-level files)
   - What are the major modules? (read_subtree on key directories)
   - What patterns exist? (read existing implementations, especially recent ones)
   - What conventions matter? (lint rules, testing patterns, naming, imports)
   - Are there relevant standards in `.admin/standards/`? (e.g., `.admin/standards/PYTHON.md` for Python, `.admin/standards/TESTING.md` for testing methodology)

2. **Define the domain vocabulary**
   - Check for `CONTEXT.md`, `AGENTS.md`, `CLAUDE.md` — these contain project-specific terminology
   - Check existing skill files in `.agents/skills/` for project knowledge
   - Use the project's existing terms. Don't introduce new names for existing concepts.

3. **Understand the request**
   - Read the user's request carefully. What is the goal, not just what they asked for?
   - Identify implicit requirements (error handling, edge cases, performance, security)
   - Identify what is explicitly NOT requested (scope boundaries)
   - Ask clarifying questions when assumptions are hidden

4. **Research external dependencies**
   - Check if libraries/frameworks are already used in the project before suggesting them
   - Read documentation for any new library before using it
   - Verify API signatures and behaviors against actual docs, not assumptions

### Checklist before writing any code:
- [ ] I understand the relevant parts of the codebase
- [ ] I know the project's conventions and patterns
- [ ] I understand what the user needs, not just what they asked for
- [ ] I can articulate the problem in one sentence
- [ ] I know what "done" looks like

---

## Phase 2: Planning

### Write a plan for anything non-trivial
If the task involves:
- More than one file change
- A new module or API
- A non-obvious design decision
- Potential for breaking existing behavior

...then write a structured plan before coding.

### A good plan contains:
1. **Goal**: One-sentence summary of what we're building
2. **Steps**: Numbered, ordered steps. Each step should be:
   - Specific: "Add `validateEmail` function to `src/utils/validation.ts`" not "Add validation"
   - Verifiable: Has a clear success criterion
   - Sized for one implement-and-verify cycle (5-15 minutes)
3. **Dependencies**: What order the steps must happen in
4. **Risk areas**: Steps that are tricky, uncertain, or likely to need iteration
5. **Verification plan**: How each step will be verified

### Plan granularity:
- **Too coarse**: "Implement the feature" — not actionable
- **Too fine**: "Import React, define props interface, write render function..." — too much overhead
- **Just right**: Steps that map to verification points (one test, one function, one file)

### Publish the plan:
Share the plan with the user. Get confirmation before executing. The user may have constraints or ordering preferences you don't know about.

---

## Phase 3: Execution

### Execute methodically

**One step at a time.** Complete one step fully before starting the next. Parallel work is tempting but creates dependency confusion and merge conflicts.

For each step:
1. **Read what exists** — the file may have changed since you last saw it
2. **Implement** — make the minimal change to satisfy the step
3. **Verify** — run the verification for this step (test, typecheck, manual check)
4. **Only then**, move to the next step
5. **Update the plan** if you discover something that changes the remaining steps

### Code change principles:
- **Surgical precision**: Change exactly what's needed, nothing more. Resist the urge to "clean up" unrelated code.
- **Pattern matching**: Follow existing patterns in the codebase. Don't introduce novel patterns for one-off needs.
- **Minimal diff**: The best code change is the one that looks obvious in the diff. If a reviewer needs explanation, simplify.
- **No speculative features**: Don't add parameters, hooks, or extensions "for future use." Add them when they're needed.

### When things go wrong:
- **Blocked?** Stop. Document what you know. Ask the user for input.
- **Unexpected complexity?** Re-plan. The original plan was based on incomplete information. Update it.
- **Found a better approach?** Share it with the user before pivoting. What seems better may have hidden tradeoffs.

---

## Phase 4: Review

### Self-review before requesting feedback:
1. **Re-read every changed file**. Does the diff make sense as a whole?
2. **Check for mistakes in editing.** Did you accidentally delete a needed line? Leave a syntax error?
3. **Verify edge cases**: Empty states, error paths, null/undefined, boundary values
4. **Check consistency**: Does the new code follow the same patterns as surrounding code?
5. **Verify tests pass**: Run the specific tests for changed code

### Types of review:

| Review type | What to check | When |
|---|---|---|
| **Spec compliance** | Does the implementation match the spec/plan? | After each implementation step |
| **Code quality** | Is the code clean, consistent, well-structured? | Before claiming completion |
| **Security** | Are there injection risks, auth bypasses, data leaks? | For auth/data-handling changes |
| **Performance** | Are there obvious perf issues (N+1, memory leaks)? | For data-intensive changes |

---

## Phase 5: Lessons

### Capture what you learned

Every development session produces knowledge. Don't let it disappear:

1. **Update or create skills** in `.agents/skills/`:
   - If you discovered a tricky gotcha in this project, update the relevant skill or the `meta` skill
   - If you learned a general pattern that applies beyond this project, create or update a skill
   - Skills must be broadly applicable — not "In this session we fixed X" but "When debugging SQL queries, check Y first"

2. **What to capture:**
   - ✅ Project-specific gotchas: "This project uses Drizzle with raw SQL for migrations — never use Drizzle Kit's push command"
   - ✅ Useful patterns: "E2E tests for CLI apps require tmux for interactive sessions"
   - ✅ Pain points resolved: "When adding new models, you must run `bun run db:generate` before `bun run db:migrate`"
   - ❌ Session specifics: "We spent 3 hours fixing the auth bug" — not useful as a skill
   - ❌ Obvious advice: "Write tests" — too generic

3. **Prune stale knowledge:**
   - If a skill becomes outdated, update or remove it
   - If a workaround is no longer needed, document that it's resolved
   - If a pattern is no longer used, remove it to prevent confusion

---

## Phase 6: Handoff

When ending a session or passing work to another context, create a handoff:

### Handoff document contents:
1. **Current state**: What was accomplished (reference plans, PRDs, ADRs by path/URL)
2. **What's left**: Remaining steps, in priority order
3. **Key decisions**: Design choices made and their rationale
4. **Gotchas encountered**: Things that would trip up the next person
5. **Suggested skills**: Which skills the next session should load
6. **Redacted sensitive info**: No API keys, passwords, or PII

### Handoff location:
Save to the OS temp directory, not the workspace. Reference it in the conversation. Use the `handoff` skill if available.

---

## Quick Reference

```
┌──────────────────────────────────────────────────────────────┐
│                  WORKFLOW LIFECYCLE                           │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Phase 1: Context ──► Phase 2: Plan ──► Phase 3: Execute     │
│     │                                      │                  │
│     └── Understand the codebase             │                  │
│     └── Define vocabulary                   │                  │
│     └── Understand request                  │                  │
│     └── Research dependencies               │                  │
│                                            │                  │
│                     ┌───────────────────────┘                  │
│                     ▼                                         │
│           Phase 4: Review ──► Phase 5: Lessons                │
│               │                           │                   │
│               └── Self-review              │                   │
│               └── Spec compliance          │                   │
│               └── Quality review           │                   │
│                                            ▼                   │
│                                  Phase 6: Handoff              │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

## Common Failure Modes

| Mode | Symptom | Antidote |
|---|---|---|
| **Jumping in** | Writing code before understanding the codebase | Phase 1 is mandatory. Don't skip. |
| **Analysis paralysis** | Endless planning without writing code | Set a timer. 10 minutes max for planning simple tasks. |
| **Scope creep** | Fixing everything you encounter | Each task has a scope. Write it down. Don't stray. |
| **Lost context** | Forgetting why a decision was made | Document as you go, not at the end. |
| **Premature optimization** | Complicating code for theoretical perf | Measure first. If you can't measure, don't optimize. |
| **Guessing APIs** | Using a library without checking docs | Read the docs. Verify the API signature. Every time. |
