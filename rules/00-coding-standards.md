# Coding Standards

## Type Hints

- Always use type hints for function parameters and return types
- Use `from __future__ import annotations` as the first import in every file
- Use `TypeVar` and `Generic` for typed generic classes
- Use `Protocol` classes for interfaces (not ABC)

## Logging

### Core Rule

- **NEVER use `print()` statements** - always use logging instead
- When encountering `print()` in existing code, replace it with appropriate logging

### Logger Setup

Every module should have a module-level logger:

```python
import logging

logger = logging.getLogger(__name__)
```

### Log Levels

Use the appropriate level for each message:

| Level | Use For |
|-------|---------|
| `logger.debug()` | Detailed diagnostic info, variable values, flow tracing |
| `logger.info()` | Normal operations, startup messages, successful completions |
| `logger.warning()` | Unexpected but handled situations, deprecations |
| `logger.error()` | Failures that prevented an operation from completing |
| `logger.exception()` | Errors with full traceback (use inside except blocks) |

### Converting Print to Logging

```python
# Bad
print(f"Processing {item}")
print(f"Error: {e}")

# Good
logger.info("Processing %s", item)
logger.error("Failed to process: %s", e)
```

### String Formatting

Use `%s` placeholders instead of f-strings for log messages:

```python
# Preferred - lazy evaluation, better performance
logger.debug("User %s performed action %s", user_id, action)

# Acceptable but less efficient
logger.debug(f"User {user_id} performed action {action}")
```

### Structured Context

Include relevant context in log messages:

```python
logger.info("Request completed", extra={"user_id": user_id, "duration_ms": duration})
logger.error("API call failed: %s", error, extra={"endpoint": url, "status": status_code})
```

## Imports

- Use absolute imports from the package root
- Sort imports with isort (configured via ruff)
- Group imports: stdlib, third-party, local

## Data Models

- Use Pydantic v2 for data models and validation
- Define models with proper field types and validators
- Use `model_dump()` instead of deprecated `dict()`

## Python Version

- Target Python 3.10+ (match statements are OK)
- Use modern syntax: `list[str]` instead of `List[str]`
- Use `|` for union types: `str | None` instead of `Optional[str]`

## Async/Await

- Use async/await everywhere in async codebases
- No sync code in async adapters or handlers
- Use `asyncio.gather()` for concurrent operations
- Use `AsyncMock` for testing async methods

## Documentation

- Use Context7 MCP to fetch up-to-date documentation when needed
- Follow existing patterns in the codebase for new code
