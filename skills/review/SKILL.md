---
name: review
description: Review code for correctness, quality, and completeness. Enforces the iron law — no completion claims without fresh verification evidence. Covers the full lifecycle: verification gate, self-verification, code review process, completion protocol, and review responses.
---

# Review

Review code changes like a quality-assurance specialist. Focus on whether the implementation is correct, complete, maintainable, and consistent with the surrounding codebase.

## The Iron Law

**NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE.**

You may not state that something works, is done, passes, or is correct without independently verifying it in the current session. Trusting cached results, assuming previous runs are still valid, or believing an agent report without checking — all are prohibited.

This is not about distrust. It's about the physics of software: code changes, environments drift, and assumptions rot. The only truth is the output of a command you just ran.

---

## When to Use

- After implementation, before claiming completion
- Before merge (if using version control) or before declaring work done
- Whenever a change needs a structured review
- Any time you need to assert "the tests pass", "the build succeeds", "the feature works", or "the bug is fixed"

---

## Part 1: The Verification Gate

When you need to assert any claim about the state of the code, you MUST pass through the verification gate first.

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

Verify as you go, not just at the end. For each implementation step:

1. **Before writing code**: Define how you'll verify success. What command proves this step works?
2. **After writing code**: Run the verification immediately. Don't accumulate unverified steps.
3. **After changes to tests or config**: Re-run everything affected. A config change can break working tests.

### Code change verification matrix:

| Change type | Minimum verification |
|---|---|
| New function | Unit test passes + typecheck |
| Bug fix | Regression test passes + original reproduction works |
| Refactor | All existing tests pass + no type errors |
| Config change | Build succeeds + affected commands work |
| Dependency update | Build succeeds + tests pass + manual smoke test |
| API change | Consumer integration test passes + docs updated |

---

## Part 3: Review Focus & Process

### Review Focus

Check the change for:

* **Correctness** — Does it behave as intended?
* **Quality** — Is it clear, maintainable, and appropriately scoped?
* **Pattern Adherence** — Does it match existing conventions?
* **Completeness** — Were all plan items implemented?
* **Test Coverage** — Are the right tests present or updated?

### Review Process

1. Read the original plan (if any) and the implementation.
2. Inspect the changes — review the diff or compare current state against previous state to confirm what changed.
3. Compare the change with existing project patterns.
4. Look for logic errors, edge cases, runtime risks, and missing tests.
5. Verify that the work is complete and that evidence is sufficient.

### Review Checklist

- [ ] Code follows project conventions
- [ ] Error handling is appropriate
- [ ] No obvious bugs or missed edge cases
- [ ] Documentation is adequate
- [ ] Tests are included or updated where needed
- [ ] Performance or maintainability concerns are addressed

### Severity Levels

* **BLOCKER** — Must fix before proceeding
* **MAJOR** — Should fix; impacts quality or correctness
* **MINOR** — Nice to have; cosmetic or low-risk

### Guardrails

* Be specific about what is wrong and why it matters.
* Do not rewrite the implementation.
* Do not approve without reviewing the full set of changes.
* Always note at least one positive finding when possible.

---

## Part 4: Review Output Format

When writing a structured review, use this template:

```markdown
## Review: [Implementation Name]

### Summary
- **Status**: [PASS / NEEDS_FIX / CRITICAL_ISSUES]
- **Files Reviewed**: [count]
- **Issues Found**: [count critical, count minor]

### Critical Issues (MUST FIX)
1. **[Issue Title]**
   - **Location**: [file:line]
   - **Problem**: [description]
   - **Impact**: [what could go wrong]
   - **Fix**: [specific recommendation]

### Minor Issues (SHOULD FIX)
1. **[Issue Title]**
   - **Location**: [file:line]
   - **Suggestion**: [improvement]

### Pattern Mismatches
1. **[Pattern]**
   - **Expected**: [existing pattern]
   - **Found**: [what was done]
   - **Recommendation**: [how to align]

### Positive Findings
- [What was done well]

### Verification Check
□ All plan items complete
□ Evidence provided and sufficient
□ Tests pass
□ No critical issues
□ Pattern adherence good

### Recommendation
[APPROVE / APPROVE_WITH_NOTES / NEEDS_REVISION]
```

---

## Part 5: Finishing & Completion

### Pre-completion check

Before declaring work done:

1. **All tests pass** — run the full test suite, not just your new tests
2. **Typecheck passes** — run the type checker
3. **Build succeeds** — run the build command
4. **No debug artifacts** — grep for console.log, debugger statements, TODO markers you added
5. **No unintended changes** — review every changed file to confirm only intentional modifications

### If using version control (e.g. git):

- **Review the diff**: Inspect `git diff` and review every changed line for unintended changes.
- **Check recent changes**: `git log --oneline -20`, `git diff HEAD~5` — often the source of a regression was introduced recently.
- **Logical commits**: Each commit should be a single, bisectable change. Not "work in progress" or "fix stuff".
- **Descriptive messages**: Explain WHAT and WHY, not just the diff. "Fix race condition in token refresh (root cause: cache invalidation timing)" not "fix bug".
- **Don't commit generated files**: Build outputs, compiled binaries, lockfile changes from unrelated dependencies.

If not using version control, simply ensure that the pre-completion checklist above is satisfied and that the work area is clean.

---

## Part 6: Requesting Code Review

When your work is ready for review, follow this protocol.

### Prepare the review:
1. **Identify the scope**: What changes need review? If using git, gather the SHAs or diff. Otherwise, identify the files that were modified and the reason.
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

## Part 7: Receiving Code Review

When you are on the receiving end of a code review (as the agent being reviewed):

### The response pattern:
1. **Read** the feedback fully before responding
2. **Understand** — restate the feedback in your own words to confirm
3. **Verify** — check if the suggestion is correct against codebase reality
4. **Evaluate** — is this a blocking issue, an improvement, or a preference?
5. **Respond** — with technical reasoning, not social performance
6. **Implement** — one item at a time, verifying each

### Forbidden responses:
- ❌ Performative agreement: "Great point!", "You're absolutely right!" — state what you changed, don't perform enthusiasm
- ❌ Defensive reactions: "Actually, I think..." before understanding the feedback
- ❌ Over-apology: "Sorry about that, my bad entirely" — fix the issue, don't dwell on blame

> **Allowed:** Brief acknowledgment paired with action (e.g., "Good catch — fixed the null check"). The goal is concise technical communication, not social performance.

### When feedback is unclear:
Stop and ask for clarification before implementing anything. Partial understanding leads to incorrect implementations. A clarifying question is faster than a wrong fix.

---

## Part 8: Verification After External Changes

When the codebase changes outside your control (e.g., rebase, merge, dependency update — if using git; or any other external modification to files):

1. **Re-verify everything**. Previous test results are invalidated.
2. **Check for conflicts** — if using version control, merge/rebase conflicts may hide semantic conflicts. Even without version control, verify that no file changes caused unintended interference.
3. **Run the full test suite**, not just your changed areas.
4. **Smoke test affected features** — a passing test suite doesn't mean the app works.

---

## Quick Reference

### Workflow Card

```
┌─────────────────────────────────────────────────────────────┐
│                     REVIEW WORKFLOW                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────┐           │
│  │         THE VERIFICATION GATE                 │           │
│  │  Identify command → Run fresh → Read output   │           │
│  │  → Verify claim → Only then, make the claim   │           │
│  └──────────────────────────────────────────────┘           │
│                                                              │
│  Self-Verification → Structured Review → Completion Check    │
│                           │                                  │
│                           ▼                                  │
│             Request Review → Receive Review                  │
│                                                              │
│  After external changes: re-verify everything                │
│                                                              │
│  "I think it works" = "I haven't verified yet"              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

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
