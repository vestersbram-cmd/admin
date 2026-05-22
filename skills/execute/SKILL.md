---
name: execute
description: Implement features or fixes with disciplined, evidence-based execution that matches existing codebase patterns.
---
# Execute

Implement the requested feature or fix with careful attention to the existing codebase.

## When to Use

- After planning is complete
- When the implementation approach is already defined
- For changes that should match existing patterns closely

## Execution Mindset

- Read the full plan before changing anything.
- Work one logical unit at a time.
- Match surrounding code style, naming, and structure.
- Verify every meaningful change before moving on.
- Treat evidence as part of the work, not an afterthought.

## Workflow

1. Review the plan and identify all tasks.
2. Inspect similar code or tests before editing.
3. Make surgical changes that stay aligned with the project.
4. Verify the change with diagnostics or tests.
5. Record what changed and what proves it works.

## Required Evidence

- File edit: diagnostics or equivalent validation is clean.
- New function or class: signature, structure, and docstring are present.
- Bug fix: include a regression test or a clear reproduction check.
- Test update: tests run and pass, or any failures are documented.
- Refactor: behavior stays intact and verification still passes.

## Output Format

### Summary
Implementation summary — what changed and why.

### Changes
Describe each significant change made, including files modified.

### Commands Run
- Commands executed during implementation (tests, typechecks, builds, etc.)

### Verification
How the change was checked and what the results were.

## Guidelines

- Follow existing code patterns exactly unless the plan says otherwise.
- Add appropriate error handling.
- Include docstrings and comments where they add clarity.
- Run verification commands before finishing.
- Update documentation when the behavior or usage changes.

## Before Starting

- Review the full plan if one exists.
- Locate 2–3 similar implementations or tests.
- Check `.admin/standards/` for relevant coding conventions (e.g., `.admin/standards/PYTHON.md` for Python, `.admin/standards/FASTAPI.md` for FastAPI).
- Note dependencies, required outputs, and evidence criteria.
- Identify any files that should be updated together.

## Completion Check

- All tasks in the plan are addressed.
- All changed files have been verified.
- Tests or diagnostics are clean, or known failures are documented.
- No new TODO/FIXME markers were introduced.
- The result is complete enough to hand off with confidence.
