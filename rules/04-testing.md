# Testing Conventions

## Test Framework

- Use pytest as the test framework
- Use pytest-asyncio for async tests

## Running Tests

```bash
# Run unit tests
uv run pytest tests/ --ignore=tests/integration/ -v

# Run single test by name
uv run pytest tests/ -k "test_name"

# Run with coverage
uv run pytest tests/ --ignore=tests/integration/ --cov=src/<package>

# Run integration tests (requires API credentials)
uv run pytest tests/integration/ -v -s --no-cov
```

## Async Tests

- Use `@pytest.mark.asyncio` decorator for async tests
- Use `AsyncMock` for mocking async methods
- Use `MagicMock` for sync methods

```python
import pytest
from unittest.mock import AsyncMock, MagicMock

@pytest.mark.asyncio
async def test_async_function():
    mock_client = AsyncMock()
    mock_client.fetch.return_value = {"data": "value"}

    result = await some_async_function(mock_client)

    assert result == expected
    mock_client.fetch.assert_called_once()
```

## Test Organization

```
tests/
├── conftest.py         # Shared fixtures
├── fixtures.py         # Test data factories
├── unit/               # Unit tests (mocked dependencies)
├── integration/        # Real API tests (skipped in CI)
└── e2e/                # End-to-end tests
```

## Fixtures

- Define shared fixtures in `conftest.py`
- Use `MockDataFactory` or similar for creating test data
- Scope fixtures appropriately (function, class, module, session)

```python
# conftest.py
import pytest
from unittest.mock import AsyncMock

@pytest.fixture
def mock_client():
    return AsyncMock()

@pytest.fixture
def sample_message():
    return {"role": "user", "content": "Hello"}
```

## Test Data Factory Pattern

Create a factory class for generating consistent test data:

```python
# tests/fixtures.py
from dataclasses import dataclass

@dataclass
class MockDataFactory:
    @staticmethod
    def make_message(role: str = "user", content: str = "test"):
        return {"role": role, "content": content}

    @staticmethod
    def make_tool_call(name: str = "test_tool", args: dict | None = None):
        return {"name": name, "arguments": args or {}}

    @staticmethod
    def make_event(event_type: str = "message", **kwargs):
        return {"type": event_type, **kwargs}
```

Use in tests:

```python
from tests.fixtures import MockDataFactory

def test_message_handling():
    message = MockDataFactory.make_message(role="assistant", content="Hello")
    # test code
```

## Skip Markers

Use markers to categorize tests:

```python
@pytest.mark.requires_api
async def test_real_api_call():
    """Skipped in CI, requires real API credentials."""
    pass

@pytest.mark.slow
def test_performance():
    """Skipped by default, run with --slow flag."""
    pass
```

Configure in `pyproject.toml`:
```toml
[tool.pytest.ini_options]
markers = [
    "requires_api: marks tests as requiring real API (skipped in CI)",
    "slow: marks tests as slow (skipped by default)",
]
```

## Test Performance

- Override slow defaults for faster tests (e.g., `_max_iterations = 3`)
- Use `pytest-xdist` for parallel test execution when needed
- Keep unit tests fast (< 100ms each)

## Integration Tests

- Store test credentials in `.env.test`
- Never commit real credentials
- Skip integration tests in CI by default
- Use separate test accounts/environments
