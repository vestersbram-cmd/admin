---
name: verify
description: Use when verifying work before claiming completion, reviewing code, or finishing a development branch. Enforces the iron law — no completion claims without fresh verification evidence. Includes self-verification, code review protocols, and quick command reference.
---
# Verify: Evidence Before Claims

## The Iron Law

**NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE.**

You may not state that something works, is done, passes, or is correct without independently verifying it in the current session. Trusting cached results, assuming previous runs are still valid, or believing an agent report without checking — all are prohibited.

This is not about distrust. It's about the physics of software: code changes, environments drift, and assumptions rot. The only truth is the output of a command you just ran.

---

## Part 1: The Verification Gate

When you need to assert any of the following, you MUST pass through the verification gate first:

- "The tests pass"
- "The build succeeds"
- "The feature works"
- "The bug is fixed"
- "The refactor is safe"
- "The deployment is ready"

### The gate:

```
1. Identify the command ─► 2. Run the command ─► 3. Read the output
       │                                                    │
       └──────────────────── 4. Verify ─────────────────────┘
                                    │
                         ┌──────────┴──────────┐
                         ▼                     ▼
                   Claim is valid         Claim is invalid
```

### Gate rules:
1. **Identify**: What specific command proves the claim? (e.g., `bun run test:types`, `bun run build`, `node repro.js`)
2. **Run**: Execute it fresh. Not "it worked earlier" — run it now.
3. **Read**: Examine the full output. Exit code 0 + no stderr is not enough — check for actual pass indicators, not just absence of errors.
4. **Verify**: Does the output definitively confirm the claim? Partial checks are insufficient. "Build succeeded" requires `exit 0` and no errors. "Tests pass" requires seeing the full green run in current logs.
5. **Claim**: Only after verification is confirmed may you state the result.

### Prohibited claims without fresh verification:
- ❌ "All tests pass" — you saw them pass 10 minutes ago, not now
- ❌ "The build should be fine" — should is not evidence
- ❌ "It worked in my last session" — sessions are independent
- ❌ "The agent report says it works" — verify the output yourself
- ❌ "No news is good news" — commands can silently fail

---

## Part 2: Self-Verification During Implementation

### Verify as you go, not just at the end

For each implementation step:

1. **Before writing code**: Define how you'll verify success. What command proves this step works?
2. **After writing code**: Run the verification immediately. Don't accumulate unverified steps.
3. **After changes to tests or config**: Re-run everything affected. A config change can break working tests.

### Code change verification checklist:

| Change type | Minimum verification |
|---|---|
| New function | Unit test passes + typecheck |
| Bug fix | Regression test passes + original reproduction works |
| Refactor | All existing tests pass + no type errors |
| Config change | Build succeeds + affected commands work |
| Dependency update | Build succeeds + tests pass + manual smoke test |
| API change | Consumer integration test passes + docs updated |

---

## Part 3: Finishing a Development Branch

When completing a feature branch, follow this completion protocol:

### Pre-completion check:
1. **All tests pass** — run the full test suite, not just your new tests
2. **Typecheck passes** — `bun run typecheck` or equivalent
3. **Build succeeds** — `bun run build` or equivalent
4. **No debug artifacts** — grep for console.log, debugger statements, TODO markers you added
5. **No unintended changes** — `git diff` and review every changed line

### Commit hygiene:
- **Logical commits**: Each commit should be a single, bisectable change. Not "work in progress" or "fix stuff".
- **Descriptive messages**: Explain WHAT and WHY, not just the diff. "Fix race condition in token refresh (root cause: cache invalidation timing)" not "fix bug".
- **Don't commit generated files**: Build outputs, compiled binaries, lockfile changes from unrelated dependencies.

---

## Part 4: Requesting Code Review

> **Note:** Parts 4-5 cover the full code review lifecycle. There is also a lightweight `review` skill at `.agents/skills/review/` for quick pre-commit self-reviews of uncommitted changes — use that when you just need a fast pass over your working tree before committing.

When your work is ready for review, follow this protocol:

### Prepare the review:
1. **Identify the scope**: What commits or changes need review? Gather the SHAs or diff.
2. **Provide context**: What does the reviewer need to know? Include:
   - The problem being solved (or link to the issue/PRD)
   - Key design decisions and why
   - What you already verified (tests, types, build)
   - What you're uncertain about (specific areas to focus on)
3. **Remove noise**: Exclude formatting-only changes, renames, or refactors that obscure the actual change.

### During review:
- **Respond to each comment specifically**. "Fixed" without explanation is not helpful.
- **Verify before implementing suggestions**. Not all review feedback is correct — check it against the codebase reality.
- **Push back when appropriate**. If a suggestion is technically incorrect, lacks context, or violates YAGNI, say so with technical reasoning.
- **Prioritize**: Blocking issues > correctness concerns > style preferences.

---

## Part 5: Receiving Code Review

When you are on the receiving end of a code review (as the agent being reviewed):

### The response pattern:
1. **Read** the feedback fully before responding
2. **Understand** — restate the feedback in your own words to confirm
3. **Verify** — check if the suggestion is correct against codebase reality
4. **Evaluate** — is this a blocking issue, an improvement, or a preference?
5. **Respond** — with technical reasoning, not social performance
6. **Implement** — one item at a time, verifying each

### Forbidden responses:
- ❌ Performative agreement: "Great point!", "You're absolutely right!"
- ❌ Gratitude expressions: "Thanks!", "Thank you!"
- ❌ Defensive reactions: "Actually, I think..."
- ❌ Over-apology: "Sorry about that, my bad entirely"

### When feedback is unclear:
Stop and ask for clarification before implementing anything. Partial understanding leads to incorrect implementations. A clarifying question is faster than a wrong fix.

---

## Part 6: Verification After External Changes

When the codebase changes outside your control (rebase, merge, dependency update):

1. **Re-verify everything**. Previous test results are invalidated.
2. **Check for conflicts** — merge/rebase conflicts may hide semantic conflicts
3. **Run the full test suite**, not just your changed areas
4. **Smoke test affected features** — a passing test suite doesn't mean the app works

---

## Appendix: Quick Verification Reference

### Common Verification Commands

Adapt these to the project's toolchain:

| Type | Example command |
|---|---|
| Type checking | `bun run typecheck` or `python -m mypy src/` |
| Linting | `bun run lint` or `ruff check src/` |
| Tests | `bun test` or `python -m pytest` |
| Format check | `bun run format:check` or `ruff format --check src/` |
| Build | `bun run build` |

### Quick Result Log Format

Use this structure to record verification results concisely:

- **Name**: typecheck — **Result**: PASS — **Command**: `bun run typecheck`
- **Name**: tests — **Result**: PASS — **Command**: `bun test`
- **Name**: build — **Result**: PASS — **Command**: `bun run build`

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────┐
│                VERIFICATION GATE                     │
├─────────────────────────────────────────────────────┤
│                                                      │
│   Before claiming:                                   │
│   1. Identify the proving command                    │
│   2. Run it fresh                                    │
│   3. Read full output                                │
│   4. Verify claim against output                     │
│   5. Only then, make the claim                       │
│                                                      │
│   After external changes: re-verify everything       │
│                                                      │
│   "I think it works" = "I haven't verified yet"     │
│                                                      │
└─────────────────────────────────────────────────────┘
```
