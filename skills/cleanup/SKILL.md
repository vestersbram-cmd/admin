---
name: cleanup
description: Simplify and clean code. Review uncommitted changes for simpler designs, reduced complexity, and better reuse.
---
# Cleanup

Review uncommitted changes (staged and unstaged) and find ways to simplify the code. Clean up logic. Find a simpler design. Reuse existing functions. Move utilities to utility files. Lower the cyclomatic complexity. Remove try/catch statements when not completely necessary.

## When to Use

- After implementation is complete, before review
- When code feels overly complex or verbose
- Before committing changes that may have accumulated technical debt
- When asked to "clean this up" or "simplify"

## Focus Areas

1. **Simplification** — Can the same result be achieved with less code? Fewer branches? Fewer variables?
2. **Reuse** — Are there existing utilities, helpers, or patterns that could replace custom code?
3. **Complexity** — Reduce nested conditionals, early returns, and cyclomatic complexity.
4. **Error Handling** — Remove unnecessary try/catch. Use explicit error returns where the codebase prefers them.
5. **Organization** — Move inline logic to named functions. Group related operations.
6. **Dead Code** — Remove unused imports, variables, functions, and commented-out code.

## Process

1. Run `git diff` to see all uncommitted changes.
2. Read each changed file and understand what it does.
3. For each complex section, ask: "Can this be simpler?"
4. Apply simplifications one at a time.
5. Verify the code still works after each change.
6. Summarize what was cleaned up.

## Output Format

### Summary
What was simplified and why.

### Changes Made
- Simplified function X by replacing Y with Z
- Moved utility logic to existing helpers
- Removed unused imports in file.py
- Reduced nesting in function W
