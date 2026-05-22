---
name: fix-type-ignores
description: Identify, analyze, and fix unnecessary or outdated type ignore comments in Python packages to improve type safety.
---
# Fix Type Ignores

## Purpose
Identify, analyze, and fix unnecessary or outdated `# type: ignore` comments in Python code. These comments suppress mypy type checking and should be used sparingly and removed when the underlying type issues are resolved.

## When to Use
- When a Python package has accumulated many `# type: ignore` comments
- After upgrading dependencies that may have improved type stubs
- When refactoring code to improve type safety
- As part of code quality maintenance and cleanup

## Usage

### Command/Action
Use grep to find all `# type: ignore` occurrences, then systematically review and fix each one:

```bash
# Find all type: ignore comments in a package
grep -r "# type: ignore" src/ --include="*.py"
```

### Analysis Process
1. **Identify all type ignores**: Search the package for `# type: ignore` comments
2. **Categorize by reason**: Note the reason specified (e.g., `# type: ignore[attr-defined]`)
3. **Attempt removal**: Remove the comment and run mypy to see if the issue still exists
4. **Fix underlying issues**: If mypy errors remain, fix the actual type problem
5. **Document necessary ignores**: For truly unavoidable ignores, add comments explaining why

### Example Workflow
```python
# Before (with unnecessary ignore)
def get_value(x: Any) -> str:
 return x # type: ignore

# After (fixed with proper typing)
def get_value(x: str) -> str:
 return x
```

## Common Pitfalls
- **Blind removal**: Don't remove type ignores without testing - the underlying issue may still exist
- **Ignoring specific error codes**: When using `# type: ignore[error-code]`, ensure you're addressing the right issue
- **Missing context**: Type ignores without comments make future maintenance difficult
- **Overlooking imports**: Some type ignores are needed for third-party libraries without proper stubs

## Testing
After fixing type ignores, verify with:
```bash
# Run mypy on the package
mypy src/

# Run with strict mode to catch more issues
mypy src/ --strict
```

## Best Practices
- Add comments explaining why a type ignore is necessary when it truly is
- Prefer fixing the underlying type issue over adding ignores
- Use specific error codes when possible: `# type: ignore[attr-defined]` instead of just `# type: ignore`
- Periodically review type ignores as dependencies and type stubs improve
