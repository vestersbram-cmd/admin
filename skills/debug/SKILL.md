---
name: debug
description: "Use when investigating bugs, performance regressions, or unexpected behavior. Follows a disciplined loop: build a feedback loop, reproduce, hypothesize, instrument, fix with a regression test, and clean up."
---

# Debug: Systematic Debugging Workflow

## The Iron Law

**NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST.**

Symptom-focused fixing is a failure mode. Every bug has a root cause. Your job is to find it before you touch any code. If you're tempted to skip investigation because it's "just a quick fix" or "obvious" — stop. That's exactly when systematic debugging matters most.

---

## Phase 1: Build a Feedback Loop

This is the single most important phase. Without a fast, deterministic, reproducible signal, you are flying blind.

### Before you do anything else:

1. **Create a minimal reproduction** — a failing test, an HTTP script, a CLI invocation, a throwaway harness. It must be:
   - **Fast**: Runs in under 5 seconds
   - **Deterministic**: Same result every time on the same input
   - **Reproducible**: Works on a clean checkout
   - **Minimal**: Stripped of everything not needed to trigger the bug

2. **The feedback loop IS the debugger.** The faster you can go from "make a change" to "see if it helped," the more hypotheses you can test per minute. Your goal is to maximize hypothesis tests per minute.

3. **If you cannot build a feedback loop:** Stop. Tell the user clearly. List what you tried. Ask for specific diagnostic data (logs, stack traces, steps to reproduce, environment details).

### Script harnesses

```bash
# Prefer a simple throwaway script over full test suite:
bun run scripts/repro.ts

# Or a one-liner:
node -e "require('./src/index').doThing()"
```

---

## Phase 2: Reproduce

Before investigating, verify that you can observe the exact bug using your feedback loop.

- Run the reproduction **before** reading code. The symptom you see tells you where to look.
- Make sure the failure matches what the user described. If it doesn't, you may be investigating the wrong problem.
- Check **recent changes** — `git log --oneline -20`, `git diff HEAD~5` — often the bug was introduced recently.

---

## Phase 3: Hypothesize

Generate 3-5 ranked, falsifiable hypotheses **before** touching any code.

### A good hypothesis is:
- **Specific**: "The `validateToken` function throws because `req.headers.authorization` is `undefined` on this endpoint" — not "auth is broken"
- **Falsifiable**: Can be proven wrong by a single experiment
- **Ranked**: In order of likelihood, with the reasons why

### The hypothesis checklist:

| Criterion | Example |
|---|---|
| Specific location | "in `src/auth/middleware.ts:42`" |
| Specific mechanism | "because `split(' ')` fails when the header is missing" |
| Observable prediction | "adding a guard for undefined will fix the crash" |
| If wrong, what it tells us | "if not, the problem is upstream in the routing layer" |

### Share hypotheses with the user

Before instrumenting, share your ranked list. The user may have domain knowledge that reorders them or eliminates some entirely. This saves enormous time.

---

## Phase 4: Instrument

Test your hypotheses one at a time, changing exactly one variable per test.

### Probes:
- **Debug logs**: Add uniquely tagged log lines (`[DEBUG-SESSION-<random>]: ...`) so they're easy to find and clean up
- **Console.log**: For Node.js debugging
- **Debugger breakpoints**: For stepping through logic
- **Performance measurements**: `console.time()` / `performance.now()` for perf regressions
- **Type assertions**: Add runtime checks to verify type assumptions

### Rules:
- **One variable at a time.** If you change two things, you don't know which one fixed it.
- **Tag all debug output** with a unique marker for cleanup.
- **Write down what you learn**, even negative results. "X is NOT the cause" is valuable data.

---

## Phase 5: Fix + Regression Test

### The fix must:
1. **Solve the root cause**, not the symptom. If you fixed how the error is displayed but not why the data is missing — you didn't fix the bug.
2. **Include a regression test** that exercises the exact bug pattern. This test should:
   - Be written at a "correct seam" — a boundary in the code where behavior can be verified without testing implementation details
   - Fail before your fix, pass after
   - Be minimal: test only the bug, not everything around it
3. **Be verified against the original reproduction** — the real-world scenario, not just the minimal test. Some bugs only manifest with specific data shapes or timing.

### If 3+ fixes fail:
Stop and reconsider the architecture. You may be fighting a design problem that no single fix can address. `design-thinking` may be more appropriate than `debug` at this point.

---

## Phase 6: Cleanup + Post-mortem

### Remove all debug artifacts:
```
find . -name "*.ts" -exec grep -l "\[DEBUG-SESSION-" {} \;
```
Remove every tagged log line, console statement, and throwaway file.

### Verify the final state:
1. Regression test passes — independently, not just part of a full suite run
2. Original full scenario works — not just the minimal reproduction
3. No debug artifacts remain

### Document the learning:
- Write the correct hypothesis in the commit message: "Root cause: ... Fixed by: ..."
- Consider: **Could an architectural improvement have prevented this bug?** If yes, create a follow-up task or skill entry. The best bugs are the ones that never happen again.

---

## Common Failure Modes

| Mode | Symptom | Antidote |
|---|---|---|
| **Guess-and-check thrashing** | Making random changes hoping something sticks | Build a feedback loop first. Always. |
| **Scope creep** | Investigating unrelated issues found along the way | Stay focused on the specific symptom. Create separate tasks. |
| **Premature fix** | Fixing before understanding root cause | Phase 3 first. Write the hypothesis down. |
| **Confirmation bias** | Only looking for evidence that supports your leading hypothesis | Actively try to disprove each hypothesis. Rank them. |
| **Complex debugging** | Diving into implementation instead of checking inputs/outputs at boundaries | Check component boundaries first. Work inward. |
| **Lost context** | Debug output mixed with production logs | Tag all debug output uniquely. Always clean up. |

---

## Quick Reference

1. **Feedback loop** → minimal, fast, deterministic reproduction
2. **Reproduce** → verify the exact symptom
3. **Hypothesize** → 3-5 ranked, falsifiable hypotheses
4. **Instrument** → one variable at a time, tagged probes
5. **Fix + test** → regression test at the correct seam
6. **Cleanup** → remove debug artifacts, verify, document
