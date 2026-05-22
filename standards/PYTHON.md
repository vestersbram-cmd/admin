# Python Style Guide

## File Structure

### Top-Level Structure
```
project_name/
├── .env              # Env vars (optional, gitignored)
├── .gitignore        # Git ignore rules
├── pyproject.toml    # Metadata, dependencies (PEP 518/621)
├── README.md         # Description, setup, usage
├── requirements.txt  # Pinned deps (if not using lock files)
├── setup.py          # Legacy packaging (if needed)
├── src/              # Main source code
└── tests/            # Test suite
```

### Source Directory (`src/`)
```
src/
└── package_name/
    ├── __init__.py
    ├── main.py         # App entry point (if any)
    ├── cli.py          # CLI (if any)
    ├── config.py       # Config loading
    ├── models.py       # Data models
    ├── services/       # Business logic
    │   ├── __init__.py
    │   └── example_service.py
    ├── utils/          # Utility functions
    │   ├── __init__.py
    │   └── helpers.py
    └── api/            # API code (e.g., FastAPI routers)
        ├── __init__.py
        └── endpoints.py
```

### Test Directory (`tests/`)
- Mirror `src/` structure for discoverability.
```
tests/
├── _package_name/    # Note the underscore prepended
│   ├── test_config.py
│   ├── services/
│   │   └── test_services_example_service.py
└── conftest.py
```

## Formatting

- Follow PEP 8
- Indent: 4 spaces (no tabs)
- Max line length: 100 chars
- Naming: `snake_case` (funcs, vars), `PascalCase` (classes), `UPPER_SNAKE_CASE` (consts)
- Strings: Double quotes, triple double quotes (`"""docstring"""`) for docstrings
- Use type hints consistently across all files.

## Forbidden Names

- Never name a folder, file, class, method or variable any of the following forbidden names (substrings are allowed):
  - sync
  - async
  - test
  - tests
  - shell
  - common libs: json, math, etc.

## Code Organization

- Keep __init__.py files empty (only a single line doc-string)
- Keep files concise, ≤12,000 characters (12 KB)
- Write small functions, ≤30 lines (excluding doc-string)
- Make files DRY (on intra- and inter-file level): Don't repeat yourself
- Order imports: stdlib, third-party, local (alphabetically within groups)

## Documentation

- Ensure 1 line docstrings per method/function/class.
- Modules: Multi-line docstring at the top describing purpose (no empty lines).

## Async

- Use below snippet in async implementations:

```python
import asyncio
if sys.platform == "win32":
    from asyncio.windows_events import WindowsProactorEventLoopPolicy
    asyncio.set_event_loop_policy(WindowsProactorEventLoopPolicy())
```

Async can add overhead from context switching. Optimize by:

- Avoiding blocking operations in async functions.
- Using `asyncio.gather()` for concurrent coroutines.
- Handling CPU-bound tasks with `asyncio.to_thread()` or multiprocessing.

## Preferred Packages

Use below packages for specific purposes:

| Purpose              | Package(s)                 |
|:-------------------- |:-------------------------- |
| API                  | fastapi                    |
| ASGI Server          | uvicorn                    |
| Async                | asyncio, aiohttp, aiofiles |
| Chrome Browser API   | browserapi (custom)        |
| Data validation      | pydantic2                  |
| Env Vars             | python-dotenv              |
| HTML Processing      | beautifulsoup4             |
| JSON                 | orjson                     |
| TOML read            | tomllib                    |
| TOML write           | tomlkit                    |
| PostgreSQL           | asyncpg                    |
| SSH                  | paramiko                   |
| Templating           | jinja2                     |
| Testing              | pytest, pytest-cov         |
| Web Requests         | httpx                      |
| WebSocket            | websockets                 |
| OS Window Management | PyWinCtl                   |

## Common Mistakes and Correct Implementation

**CRITICAL: Always implement the optimal approach to avoid performance issues.**

### ❌ 1. DON'T: Use nested loops for finding common elements

```python
# O(n²) - SLOW
for item1 in list1:
    for item2 in list2:
        if item1 == item2: common.append(item1)
```

### ✅ 1. DO: Use sets for O(1) lookups

```python
# O(n) - FAST
set2 = set(list2)
common = [item for item in list1 if item in set2]
```

---

### ❌ 2. DON'T: Use lists for membership testing

```python
if item in large_list:  # O(n) - SLOW
```

### ✅ 2. DO: Use sets for membership testing

```python
if item in large_set:  # O(1) - FAST
```

---

### ❌ 3. DON'T: Load huge datasets into memory

```python
large_list = [i for i in range(1_000_000_000)]  # MemoryError
```

### ✅ 3. DO: Use generators for large data

```python
large_gen = (i for i in range(1_000_000_000))  # Memory efficient
```

---

### ❌ 4. DON'T: Use lists for key-value lookups

```python
for key, value in data_list:
    if key == target: return value  # O(n) - SLOW
```

### ✅ 4. DO: Use dictionaries for key-value lookups

```python
return data_dict[target]  # O(1) - FAST
```