# Standard Project File Structure

Defines a consistent layout for Python projects.

## Example Top-Level Structure
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

## Example Source Directory (`src/`)
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

## Example Test Directory (`tests/`)
- Mirror `src/` structure for discoverability.
```
tests/
├── _package_name/    # Note the underscore prepended
│   ├── test_config.py
│   ├── services/ 
│   │   └── test_services_example_service.py
└── conftest.py
```
