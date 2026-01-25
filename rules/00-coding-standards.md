# Coding Standards

## Type Hints

- Always use type hints for function parameters and return types
- Use `from __future__ import annotations` as the first import in every file
- Use `TypeVar` and `Generic` for typed generic classes
- Use `Protocol` classes for interfaces (not ABC)

## Logging

- NEVER use `print()` statements - always use logging instead
- Use module-level logger: `logger = logging.getLogger(__name__)`
- Log full error details at appropriate levels (debug/info/warning/error)

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
