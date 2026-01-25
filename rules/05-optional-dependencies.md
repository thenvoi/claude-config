# Optional Dependencies Pattern

## Overview

Use optional dependencies to keep the core package lightweight while supporting multiple frameworks.

## pyproject.toml Configuration

```toml
[project.optional-dependencies]
langgraph = ["langgraph", "langchain-openai"]
anthropic = ["anthropic"]
crewai = ["crewai"]
dev = ["pytest", "pytest-asyncio", "ruff", "pyrefly"]
```

## Lazy Imports in Adapters

Adapters should import framework dependencies lazily to avoid import errors when the optional dependency is not installed.

```python
# Good - lazy import inside the adapter
class LangGraphAdapter:
    def __init__(self):
        try:
            from langgraph.graph import StateGraph
        except ImportError:
            raise ImportError(
                "langgraph is required for LangGraphAdapter. "
                "Install with: pip install thenvoi[langgraph]"
            )
        self.graph = StateGraph()

# Bad - top-level import
from langgraph.graph import StateGraph  # Fails if langgraph not installed

class LangGraphAdapter:
    pass
```

## Testing with Optional Dependencies

Skip tests when optional dependencies are missing:

```python
import pytest

try:
    import langgraph
    HAS_LANGGRAPH = True
except ImportError:
    HAS_LANGGRAPH = False

@pytest.mark.skipif(not HAS_LANGGRAPH, reason="langgraph not installed")
def test_langgraph_adapter():
    from mypackage.adapters.langgraph import LangGraphAdapter
    # test code
```

Or use a marker:

```python
requires_langgraph = pytest.mark.skipif(
    not HAS_LANGGRAPH,
    reason="langgraph not installed"
)

@requires_langgraph
def test_langgraph_feature():
    pass
```

## Installation Commands

```bash
# Install core only
uv add thenvoi

# Install with specific adapter
uv add thenvoi[langgraph]
uv add thenvoi[anthropic]

# Install with multiple adapters
uv add thenvoi[langgraph,anthropic]

# Install with dev dependencies
uv add thenvoi[dev]
```
