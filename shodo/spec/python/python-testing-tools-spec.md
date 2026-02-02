# Python Testing Tools Specification

## Overview

Specification for Python testing tools: pytest, hypothesis, coverage, and related tooling.

## pytest Configuration

### Project Configuration
**Requirement**: Configure pytest in `pyproject.toml`.

**Mandatory Settings:**
```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
```

### Recommended Options
```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = [
    "--strict-markers",      # Fail on unknown markers
    "--cov=src",            # Coverage for src directory
    "--cov-branch",         # Branch coverage
    "--cov-report=term-missing",  # Show missing lines
    "--cov-fail-under=90",  # Minimum 90% coverage
    "--tb=short",           # Short traceback format
    "--maxfail=5",          # Stop after 5 failures
    "-v",                   # Verbose output
    "-ra",                  # Show summary of all test outcomes
]
```

### Test Markers
**Requirement**: Define custom markers for test categorization.

**Mandatory Markers:**
```toml
markers = [
    "unit: Unit tests for individual functions",
    "integration: Integration tests for component interaction",
    "property: Property-based tests for invariants",
    "performance: Performance and load tests",
    "slow: Tests that take significant time to run",
]
```

**Optional Markers:**
```toml
markers = [
    "anti_hallucination: Tests to catch LLM errors",
    "database: Tests requiring database connection",
    "external: Tests requiring external services",
    "flaky: Tests known to be intermittently failing",
]
```

### Test Discovery
**Requirement**: Follow pytest test discovery conventions.

**Rules:**
- Test files: `test_*.py` or `*_test.py`
- Test classes: `Test*` (no `__init__` method)
- Test functions: `test_*`
- Test methods: `test_*` (in Test* classes)

## pytest Fixtures

### Fixture Scopes
**Requirement**: Use appropriate fixture scopes.

**Scopes:**
- `function` - Default, run once per test function
- `class` - Run once per test class
- `module` - Run once per test module
- `package` - Run once per test package
- `session` - Run once per test session

### Fixture Best Practices
**Requirement**: Follow fixture best practices.

**Rules:**
- Use `conftest.py` for shared fixtures
- Keep fixtures simple and focused
- Avoid fixture interdependencies
- Document fixture purpose clearly
- Use autouse sparingly

**Example:**
```python
# conftest.py
import pytest
from decimal import Decimal

@pytest.fixture
def sample_user():
    """Provide sample user for testing."""
    return {
        "id": 1,
        "email": "test@example.com",
        "name": "Test User"
    }

@pytest.fixture
def sample_amount():
    """Provide sample monetary amount."""
    return Decimal("100.00")
```

## pytest Parametrization

### Basic Parametrization
**Requirement**: Use `@pytest.mark.parametrize` for scenario testing.

**Pattern:**
```python
@pytest.mark.parametrize(
    "input_value,expected",
    [
        (value1, expected1),
        (value2, expected2),
        (value3, expected3),
    ],
)
def test_function(input_value, expected):
    """Test function with multiple scenarios."""
    result = function_under_test(input_value)
    assert result == expected
```

### Complex Parametrization
**Pattern:**
```python
@pytest.mark.parametrize(
    "user_type,amount,expected_discount",
    [
        ("standard", Decimal("100"), Decimal("0")),
        ("premium", Decimal("100"), Decimal("10")),
        ("premium", Decimal("1000"), Decimal("150")),
    ],
    ids=["standard-user", "premium-small", "premium-large"],
)
def test_discount_calculation(user_type, amount, expected_discount):
    """Test discount calculation for various scenarios."""
    result = calculate_discount(user_type, amount)
    assert result == expected_discount
```

## Hypothesis (Property-Based Testing)

### Installation
**Requirement**: Include hypothesis in test dependencies.

```toml
[project.optional-dependencies]
test = [
    "pytest>=8.0.0",
    "hypothesis>=6.90.0",
    "pytest-cov>=4.1.0",
]
```

### Basic Property Test
**Pattern:**
```python
from hypothesis import given
from hypothesis import strategies as st

@given(st.integers())
def test_function_with_any_integer(value):
    """Test function handles any integer correctly."""
    result = function_under_test(value)
    # Assert properties that must always hold
    assert isinstance(result, int)
```

### Common Strategies
**Requirement**: Use appropriate hypothesis strategies.

**Built-in Strategies:**
- `st.integers()` - Integer values
- `st.floats()` - Float values
- `st.text()` - Unicode text
- `st.booleans()` - Boolean values
- `st.lists()` - Lists of values
- `st.dictionaries()` - Dictionaries
- `st.tuples()` - Tuples
- `st.emails()` - Email addresses
- `st.uuids()` - UUID values

**Composite Strategies:**
```python
from hypothesis import strategies as st

# Custom strategy for user data
user_strategy = st.fixed_dictionaries({
    "id": st.integers(min_value=1),
    "email": st.emails(),
    "name": st.text(min_size=1, max_size=100),
    "is_active": st.booleans(),
})

@given(user_strategy)
def test_user_validation(user):
    """Test user validation with generated data."""
    result = validate_user(user)
    assert result.is_valid or result.errors
```

### Hypothesis Configuration
**Requirement**: Configure hypothesis for project needs.

**Configuration:**
```python
from hypothesis import settings, HealthCheck

# In conftest.py or test file
settings.register_profile(
    "ci",
    max_examples=1000,
    deadline=None,
    suppress_health_check=[HealthCheck.too_slow],
)

settings.register_profile("dev", max_examples=100)
settings.load_profile("dev")  # or "ci" in CI environment
```

## Coverage Configuration

### Coverage Settings
**Requirement**: Configure coverage in `pyproject.toml`.

**Mandatory:**
```toml
[tool.coverage.run]
source = ["src"]
branch = true
omit = [
    "*/tests/*",
    "*/test_*.py",
    "*/__pycache__/*",
    "*/venv/*",
]

[tool.coverage.report]
precision = 2
show_missing = true
skip_covered = false
fail_under = 90.0

exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
    "if __name__ == .__main__.:",
    "if TYPE_CHECKING:",
    "@abstractmethod",
]
```

### Coverage Targets
**Requirement**: Meet minimum coverage thresholds.

**Targets:**
- Overall coverage: ≥ 90%
- Branch coverage: ≥ 90%
- Business logic: 100%
- Critical paths: 100%

### Coverage Reports
**Requirement**: Generate coverage reports.

**Formats:**
- Terminal: `--cov-report=term-missing`
- HTML: `--cov-report=html`
- XML: `--cov-report=xml` (for CI)
- JSON: `--cov-report=json` (for tooling)

## pytest Plugins

### Recommended Plugins
**Requirement**: Use appropriate pytest plugins.

**Essential:**
- `pytest-cov` - Coverage integration
- `pytest-mock` - Mocking support
- `pytest-asyncio` - Async test support

**Recommended:**
- `pytest-xdist` - Parallel test execution
- `pytest-timeout` - Test timeout enforcement
- `pytest-benchmark` - Performance benchmarking
- `pytest-random-order` - Random test ordering

**Installation:**
```toml
[project.optional-dependencies]
test = [
    "pytest>=8.0.0",
    "pytest-cov>=4.1.0",
    "pytest-mock>=3.12.0",
    "pytest-asyncio>=0.21.0",
    "pytest-xdist>=3.5.0",
    "hypothesis>=6.90.0",
]
```

## Mocking and Patching

### unittest.mock
**Requirement**: Use `unittest.mock` for mocking.

**Pattern:**
```python
from unittest.mock import Mock, patch

def test_function_with_mock():
    """Test function with mocked dependency."""
    # Create mock
    mock_service = Mock()
    mock_service.fetch_data.return_value = {"key": "value"}

    # Use mock
    result = function_under_test(service=mock_service)

    # Verify mock calls
    mock_service.fetch_data.assert_called_once()
    assert result == expected_result
```

### pytest-mock
**Requirement**: Use pytest-mock for simpler mocking.

**Pattern:**
```python
def test_function_with_mocker(mocker):
    """Test function using pytest-mock."""
    # Create mock
    mock_service = mocker.patch("module.service")
    mock_service.fetch_data.return_value = {"key": "value"}

    # Use and verify
    result = function_under_test()
    mock_service.fetch_data.assert_called_once()
```

## Async Testing

### pytest-asyncio
**Requirement**: Use pytest-asyncio for async tests.

**Configuration:**
```toml
[tool.pytest.ini_options]
asyncio_mode = "auto"
```

**Pattern:**
```python
import pytest

@pytest.mark.asyncio
async def test_async_function():
    """Test async function."""
    result = await async_function_under_test()
    assert result == expected_value
```

## Test Performance

### Parallel Execution
**Requirement**: Use pytest-xdist for parallel execution.

**Command:**
```bash
pytest -n auto  # Use all CPU cores
pytest -n 4     # Use 4 workers
```

### Test Timeouts
**Requirement**: Set timeouts for long-running tests.

**Pattern:**
```python
@pytest.mark.timeout(5)  # 5 second timeout
def test_with_timeout():
    """Test with timeout."""
    result = function_under_test()
    assert result == expected_value
```

## CI/CD Integration

### GitHub Actions
**Pattern:**
```yaml
- name: Run tests
  run: |
    pytest --cov=src --cov-report=xml --cov-report=term-missing

- name: Upload coverage
  uses: codecov/codecov-action@v3
  with:
    files: ./coverage.xml
```

### Test Selection
**Requirement**: Support test selection in CI.

**Commands:**
```bash
# Run only unit tests
pytest -m unit

# Run except slow tests
pytest -m "not slow"

# Run specific test file
pytest tests/test_specific.py

# Run tests matching pattern
pytest -k "test_user"
```

## Best Practices

### Test Organization
- One test file per source file
- Group related tests in classes
- Use descriptive test names
- Keep tests focused and simple

### Test Data
- Use fixtures for shared data
- Keep test data minimal
- Avoid hardcoded values
- Use factories for complex data

### Assertions
- One logical assertion per test
- Use specific assertions (`assert x == y`, not `assert x`)
- Include failure messages when helpful
- Validate side effects explicitly

### Test Maintenance
- Keep tests DRY (Don't Repeat Yourself)
- Refactor tests like production code
- Update tests with code changes
- Remove obsolete tests
