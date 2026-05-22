---
name: review
description: Review uncommitted or staged code changes for correctness, quality, completeness, and alignment with project patterns.
---
# Review

Review code changes like a quality-assurance specialist. Start by running git commands to see the current unstaged and staged changes. Read those files and any others that are relevant. Find ways to simplify, improve the code, and catch any bugs.

Focus on whether the implementation is correct, complete, maintainable, and consistent with the surrounding codebase.

## When to Use

Use this skill after implementation, before merge, or whenever a change needs a structured review.

## Review Focus

Check the change for:

* **Correctness** — Does it behave as intended? Are there logic errors or missed edge cases?
* **Quality** — Is it clear, maintainable, and appropriately scoped?
* **Pattern Adherence** — Does it match existing conventions, naming, and architecture?
* **Completeness** — Were all plan items implemented? Is nothing left half-done?
* **Test Coverage** — Are the right tests present, updated, or missing?
* **Simplification** — Can anything be simplified, deduplicated, or made more idiomatic?
* **Bug Hunting** — Look for obvious bugs, race conditions, null-reference risks, and off-by-one errors.

## Review Process

1. **Get the diff** — Run `git diff` (or `git diff --staged`) to see what changed.
2. **Read the plan** — Review the original plan, requirements, or issue that motivated the change.
3. **Read the implementation** — Open each changed file and read the full context, not just the diff.
4. **Compare with patterns** — Check that the change aligns with existing project conventions.
5. **Hunt for issues** — Look for logic errors, edge cases, runtime risks, missing tests, and security concerns.
6. **Verify completeness** — Confirm all plan items are addressed and evidence is sufficient.
7. **Note positives** — Always identify at least one thing done well.

## Output Format

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

### Verification Checklist
- [ ] All plan items complete
- [ ] Evidence provided and sufficient
- [ ] Tests pass
- [ ] No critical issues
- [ ] Pattern adherence good

### Recommendation
[APPROVE / APPROVE_WITH_NOTES / NEEDS_REVISION]
```

## Review Checklist

* [ ] Code follows project conventions
* [ ] Error handling is appropriate
* [ ] No obvious bugs or missed edge cases
* [ ] Documentation is adequate
* [ ] Tests are included or updated where needed
* [ ] Performance or maintainability concerns are addressed
* [ ] Security considerations checked (if applicable)

## Severity Levels

* **BLOCKER** — Must fix before proceeding. Impacts correctness, security, or stability.
* **MAJOR** — Should fix; significantly impacts quality, maintainability, or correctness.
* **MINOR** — Nice to have; cosmetic, stylistic, or low-risk.

## Guardrails

* Be specific about what is wrong and why it matters.
* Do not rewrite the implementation during the review.
* Do not approve without reviewing the full diff.
* Always note at least one positive finding when possible.
* If you find a bug, explain the scenario that triggers it.
* If you suggest a pattern change, point to an existing example in the codebase.
