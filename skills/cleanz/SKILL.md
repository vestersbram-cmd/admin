---
name: cleanz
description: Run code quality checks on Python packages using the cleanz tool — check, fix violations, and verify results.
---

# Cleanz Code Quality Checker

Use this skill to check and fix code quality in Python packages. Cleanz runs linting rules (similar to Ruff, pylint, etc.) and reports actionable violations that you can fix systematically.

## When to Use

- After implementation is complete — before committing
- When code needs style fixes or has linter violations
- To find violations, style issues, or potential bugs across a codebase
- To verify no new violations were introduced after changes

## Prerequisites

- The project follows standard Python structure: `pkg_name/src/pkg_name/`
- `cleanz` must be installed — typically at `~/.local/bin/cleanz`
- The working directory should be the package root (`pkg_name/` itself)

## Usage

### Basic check

```bash
cd /path/to/package && cleanz src/<pkg_name>
```

The `pkg_name` is both the directory name and the source subfolder under `src/`. For example:

```bash
cd my_app && cleanz src/my_app
```

### If pkg_name is unknown

Run `basename "$(pwd)"` to auto-detect the package name.

### If cleanz is not in PATH

```bash
cd ~/dev/packages/cleanz && python -m src.cleanz src/<pkg_name>
```

## Interpreting Output

The output includes violations with:

| Field | Description |
|---|---|
| File path & line | Where the issue occurs |
| Rule code | e.g., COM001, DOC002, IMP003 |
| Severity | error or warning |
| Message | Description of what's wrong |
| Fixable | Whether cleanz can auto-fix it |

### Example output

```
src/my_app/main.py:42: COM001 Missing trailing comma in function call
src/my_app/main.py:55: DOC002 Missing docstring for public function
src/my_app/utils.py:12: IMP003 Wildcard import detected (fixable)
```

## Workflow: Check, Fix, Verify

Follow this process when fixing violations:

1. **Run** `cleanz src/<pkg_name>/` in terminal
2. **Review** all violations — group by severity (errors first, then warnings)
3. **Fix** each violation systematically:
   - Focus on style violations only (PEP 8, import order, naming)
   - Don't change logic unless necessary
   - Auto-fixable issues first, manual fixes second
4. **Re-run** cleanz to verify all issues resolved
5. **Update tests** if refactoring changed function signatures
6. **Report** changes — summarize what was fixed, file paths involved, and any test updates needed

When reporting findings to the user, include:
- File paths, line numbers, and messages — findings should be easy to locate
- Which issues are auto-fixable vs. manual
- Summary: total violations, fixable count, most common rule types

### Example report

```
Summary: Fixed 12 violations in src/my_app/
Changes:
- Fixed import ordering in main.py
- Added docstrings to 3 public functions
- Renamed ambiguous variables in utils.py
Commands: cleanz src/my_app/
```

## Common Pitfalls

- **"cleanz: command not found"**: Check `~/.local/bin/cleanz` is installed and in PATH, or use the fallback `python -m src.cleanz` invocation
- **"Wrong directory structure"**: Cleanz expects `pkg_name/src/pkg_name/` structure — flat packages may not work
- **"No violations found"**: This is a success — the code passed all checks
- **"Exit code non-zero even with violations"**: Expected — cleanz exits non-zero when violations are found
- **"Empty output"**: Double-check you're in the right directory and the source path is correct
