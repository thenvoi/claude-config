# Code Quality

## Linting with Ruff

Ruff is used for linting and import sorting.

```bash
# Check for linting issues
uv run ruff check .

# Auto-fix issues
uv run ruff check . --fix
```

### Common Ruff Rules

- `E` - pycodestyle errors
- `F` - Pyflakes
- `I` - isort (import sorting)
- `UP` - pyupgrade (modern Python syntax)
- `B` - flake8-bugbear (common bugs)

### Typical Ruff Configuration

```toml
# pyproject.toml
[tool.ruff]
line-length = 88
target-version = "py310"

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B"]
ignore = [
    "E501",  # Line too long (handled by formatter)
]

[tool.ruff.lint.isort]
known-first-party = ["thenvoi"]  # Replace with your package name
```

## Formatting with Ruff

```bash
# Format code
uv run ruff format .

# Check formatting without changes
uv run ruff format . --check
```

### Formatting Standards

- Line length: 88 characters (Black default)
- Use double quotes for strings
- Trailing commas in multi-line structures

## Type Checking with Pyrefly

Pyrefly is used for type checking (not mypy).

```bash
# Run type checker
uv run pyrefly check
```

### Type Checking Notes

- Fix type errors before committing
- Use `# type: ignore` sparingly with explanation
- Prefer proper typing over ignoring errors

### When to Use `# type: ignore`

Use type ignores only when:
- Third-party library has incomplete type stubs
- Dynamic behavior that can't be expressed in types
- Workaround for known type checker limitations

Always include a comment explaining why:

```python
# type: ignore[arg-type]  # langgraph returns untyped dict
result = some_dynamic_call()

# type: ignore[assignment]  # Pydantic model validates at runtime
self.value = untrusted_input
```

### Known Typing Limitations

- Some async patterns may need explicit type annotations
- Generic callbacks across frameworks may need `Any`
- Dynamic tool registration may require runtime validation

## Package Management with uv

After cloning a repository, always install dev dependencies:

```bash
# Install with dev dependencies (required for development)
uv sync --extra dev
```

Other common commands:

```bash
# Install production dependencies only
uv sync

# Add a dependency
uv add <package>

# Add optional dependency to a group
uv add --optional <group> <package>
```

## Pre-Commit Workflow

Run these before every commit:

```bash
uv run ruff check .
uv run ruff format .
uv run pyrefly check
uv run pytest tests/ --ignore=tests/integration/ -v
```
