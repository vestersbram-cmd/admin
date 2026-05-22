# Pytest Guidelines

## File Structure

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

**File and Directory Structure**

- Each source file (`<name>.py`) requires one test file (`test_<relative_path>.py`).
- Place test files in top-level `tests/`, mirroring the source code's package structure.
- Example:
  * Source: `src/my_project/services/user_service.py`
  * Test: `tests/services/test_services_user_service.py`

**Test Anatomy and Naming**

- Test functions: `test_<function_name>_<condition>_<expected_result>`.
- Test classes: `Test<ClassName>`.
- Use AAA pattern: Arrange, Act, Assert.
- Ensure tests are isolated and focused on single behaviors.

**Assertions and Error Handling**

- Use specific assertions (e.g., `assert user.name == "Alice"`).
- Test exceptions with `pytest.raises`.
- Compare floats with `pytest.approx`.

**Pytest Features**

- Fixtures: Define in `conftest.py` or test files; use appropriate scope.
- Parametrization: Use `@pytest.mark.parametrize` for multiple test cases.
- Mocking: Use `mocker` to isolate dependencies.
- Markers: Categorize tests (e.g., `@pytest.mark.slow`).
- Output capturing: Use `capsys` for functions that print.

**Mocking Timeouts, Sleeps, and Waits**

- mock_sleep: Always mock or patch `time.sleep`, `asyncio.sleep`, and any polling/waiting logic in tests to keep the suite fast and deterministic.
- patch_sleep: Use `unittest.mock.patch` or pytest fixtures to replace sleep functions with no-ops or fast mocks (e.g., `patch("asyncio.sleep", new=AsyncMock())`).
- patch_wait: For code that polls or waits for conditions (e.g., file appearance, network events), patch the relevant helpers to return immediately or simulate the desired result.
- no_real_sleep: Never allow real timeouts or sleeps in unit tests—this ensures tests run quickly and reliably in all environments.
- slow_marker: If a test requires a long-running or slow operation, mark it with `@pytest.mark.slow` and ensure it is skipped or mocked by default.

**Code Quality and Style**

- Write clear, PEP 8-compliant test code.
- Use type hints if source code does.
- Keep unit tests fast; mock heavy operations.

**Example Test File**

*Source: `my_project/math_utils.py`*

```python
def add(a: int, b: int) -> int:
    return a + b

def divide(a: float, b: float) -> float:
    if b == 0:
        raise ValueError("Cannot divide by zero.")
    return a / b
```

*Test: `tests/test_math_utils.py`*

```python
import pytest
from my_project import math_utils

@pytest.fixture
def sample_numbers() -> tuple[int, int]:
    return 10, 5

class TestDivide:
    def test_divide_valid_numbers_returns_correct_float(self):
        result = math_utils.divide(10.0, 4.0)
        assert result == 2.5

    def test_divide_by_zero_raises_value_error(self):
        with pytest.raises(ValueError, match="Cannot divide by zero."):
            math_utils.divide(10.0, 0)

@pytest.mark.parametrize("a, b, expected", [
    (1, 2, 3), (-1, -1, -2), (0, 5, 5), (-5, 5, 0),
])
def test_add_various_inputs(a, b, expected):
    assert math_utils.add(a, b) == expected
```