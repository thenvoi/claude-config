# Add Framework Integration (TDD Workflow)

You are adding a new framework adapter and history converter to the Thenvoi SDK using a test-driven development workflow. The framework name is: **$ARGUMENTS**

**Naming conventions used in this document:**
- `$ARGUMENTS` — the lowercase module name (e.g. `openai`, `gemini`)
- `{Framework}` — the PascalCase class prefix (e.g. `OpenAI`, `Gemini`). Derive this from `$ARGUMENTS`.

Follow each phase in order. Do NOT skip ahead — the conformance tests must fail before you write the implementation.

---

## Phase 1: Scaffold Source Files

Create empty/minimal source files so imports resolve.

If the framework requires an external SDK (e.g. `openai`, `google-generativeai`), add an optional dependency group in `pyproject.toml`:
```toml
[project.optional-dependencies]
$ARGUMENTS = ["<package-name>>=<min-version>"]
```

1. **Create the converter** at `src/thenvoi/converters/$ARGUMENTS.py`:
   - Import `from __future__ import annotations`
   - Define a class `{Framework}HistoryConverter` with a `convert(self, raw: list[dict[str, Any]]) -> T` method that returns the framework's empty result (e.g. `[]` or `""`)
   - Add `set_agent_name(self, name: str)` and `__init__(self, *, agent_name: str | None = None)` following the pattern in existing converters
   - Use `from thenvoi.converters._tool_parsing import parse_tool_call, parse_tool_result` for tool event parsing

2. **Create the adapter** at `src/thenvoi/adapters/$ARGUMENTS.py`:
   - Import `from __future__ import annotations`
   - Define a class `{Framework}Adapter` extending `SimpleAdapter[T]` where `T` is the converter output type
   - Include `__init__` with at minimum: `model`, `custom_section`, `enable_execution_reporting`, `history_converter`
   - Include `_custom_tools: list = []` if the framework supports custom tools
   - Stub `async def on_message(...)`, `async def on_started(...)`, `async def on_cleanup(...)` methods

---

## Phase 2: Register with Conformance Infrastructure

### 2a. Create an Output Adapter

Edit `tests/framework_configs/output_adapters.py`:

- Add a new class implementing the `OutputAdapter` protocol
- Choose the base class that matches your output format:
  - `BaseDictListOutputAdapter` — if output is `list[dict]`
  - `LangChainOutputAdapter` pattern — if output uses custom message objects
  - `StringOutputAdapter` pattern — if output is a joined string
  - `SenderDictListAdapter` pattern — if output is `list[dict]` with `sender`/`sender_type` fields
- Implement: `result_length`, `get_content`, `get_role`, `is_empty`, `content_contains`, `assert_element_type`, `assert_sender_metadata`, `assert_result_type`
- Add the class to `__all__`

### 2b. Register the Converter Config

Edit `tests/framework_configs/converters.py`:

1. Add a factory function:
   ```python
   def _$ARGUMENTS_factory(**kw: Any) -> Any:
       from thenvoi.converters.$ARGUMENTS import {Framework}HistoryConverter
       return {Framework}HistoryConverter(**kw)
   ```

2. Add a builder function `_build_$ARGUMENTS_config()` returning a `ConverterConfig` with:
   - `framework_id` matching the module filename (e.g. `"openai"`)
   - `converter_factory` pointing to the factory
   - `empty_result` — `[]` for list-based, `""` for string-based
   - `output_adapter` — instance of your output adapter
   - Behavioral flags: `filters_own_messages`, `skips_tool_events`, `empty_sender_behavior`, `missing_sender_behavior`, `skips_empty_content`, `has_role_concept`, `has_sender_metadata`, `other_agent_output_role`

3. Append the builder to `_CONVERTER_CONFIG_BUILDERS`

### 2c. Register the Adapter Config

Edit `tests/framework_configs/adapters.py`:

1. Add a factory function that handles required constructor args with mocks:
   ```python
   def _$ARGUMENTS_factory(**kw: Any) -> Any:
       from thenvoi.adapters.$ARGUMENTS import {Framework}Adapter
       # Inject mocks for required args (e.g. kw.setdefault("llm", MagicMock()))
       return {Framework}Adapter(**kw)
   ```

2. Add a builder function `_build_$ARGUMENTS_config()` returning an `AdapterConfig` with:
   - `framework_id` matching the module filename
   - `adapter_factory` pointing to the factory
   - `expected_initial_values` — use `_default_from_init(cls, param)` to read defaults from `__init__`
   - `custom_kwargs` / `custom_expected` — test values for custom initialization
   - `has_custom_tools_attr`, `custom_tools_attr`, `has_history_converter`
   - `skip_on_started_conformance=True` if `on_started` creates real external clients

3. Append the builder to `_ADAPTER_CONFIG_BUILDERS`

---

## Phase 3: Run Conformance Tests (Expect Failures)

Run tests and verify they fail for the right reasons:

```bash
# Config drift — should now PASS (module registered)
uv run pytest tests/framework_conformance/test_config_drift.py -v

# Adapter conformance — some may fail until implementation is complete
uv run pytest tests/framework_conformance/test_adapter_conformance.py -v -k "$ARGUMENTS"

# Converter conformance — some may fail until implementation is complete
uv run pytest tests/framework_conformance/test_converter_conformance.py -v -k "$ARGUMENTS"
```

Fix failures one by one by implementing the converter and adapter logic.

---

## Phase 4: Implement the Converter

In `src/thenvoi/converters/$ARGUMENTS.py`, implement `convert()`:

1. **Text messages**: Format as `[sender_name]: content` (or framework equivalent)
2. **Own agent filtering**: Skip messages where `role == "assistant"` and `sender_name == self._agent_name`
3. **Other agent messages**: Remap to user role (or keep as assistant — match your `other_agent_output_role` config)
4. **Tool events**: Convert `tool_call` / `tool_result` to the framework's tool format, or skip if `skips_tool_events=True`
5. **Thought messages**: Always skip `message_type == "thought"`
6. **Defaults**: `role` defaults to `"user"`, `message_type` defaults to `"text"`

After each change, re-run:
```bash
uv run pytest tests/framework_conformance/test_converter_conformance.py -v -k "$ARGUMENTS"
```

---

## Phase 5: Implement the Adapter

In `src/thenvoi/adapters/$ARGUMENTS.py`, implement:

1. **`on_started`**: Set `self.agent_name`, `self.agent_description`, call `self.history_converter.set_agent_name(agent_name)`, render system prompt, create framework client
2. **`on_message`**: Bootstrap room state, convert history, invoke framework LLM, process response, send messages via `tools.send_message()` / `tools.send_event()`
3. **`on_cleanup`**: Clean up per-room state (message history, sessions). Must be safe for nonexistent rooms.

After each change, re-run:
```bash
uv run pytest tests/framework_conformance/test_adapter_conformance.py -v -k "$ARGUMENTS"
```

---

## Phase 6: Write Framework-Specific Tests

### 6a. Adapter-specific tests

Create `tests/adapters/test_$ARGUMENTS_adapter.py`:

```
"""Tests for {Framework}Adapter.

Tests for shared adapter behavior (initialization defaults, custom kwargs,
history_converter, on_started agent_name/description, on_message callable,
cleanup safety) live in tests/framework_conformance/test_adapter_conformance.py.
This file contains {Framework}-specific behavior: ...
"""
```

Cover: system prompt rendering, LLM invocation, tool execution, stream handling, error handling, custom tools integration.

Use `MagicMock()` as the base for `mock_tools` fixtures with explicit `AsyncMock()` methods to avoid "coroutine was never awaited" warnings.

### 6b. Converter-specific tests

Create `tests/converters/test_$ARGUMENTS.py`:

```
"""Tests for {Framework}HistoryConverter.

Tests for shared converter behavior (user messages, agent filtering, empty
history, edge cases, output shape) live in
tests/framework_conformance/test_converter_conformance.py.
This file contains {Framework}-specific ...
"""
```

Cover: tool event conversion format, batching, multi-message joining, malformed input handling, mixed history integration.

---

## Phase 7: Final Validation

```bash
# Full conformance suite
uv run pytest tests/framework_conformance/ tests/framework_configs/ -v

# Framework-specific tests
uv run pytest tests/adapters/test_$ARGUMENTS_adapter.py tests/converters/test_$ARGUMENTS.py -v

# Full test suite
uv run pytest tests/ --ignore=tests/integration/ -v

# Lint
uv run ruff check .
uv run ruff format .
```

All tests must pass. Config drift tests must show no uncovered modules.

---

## Quick Reference: Key Files

| Purpose | Path |
|---|---|
| Adapter source | `src/thenvoi/adapters/$ARGUMENTS.py` |
| Converter source | `src/thenvoi/converters/$ARGUMENTS.py` |
| Adapter config registry | `tests/framework_configs/adapters.py` |
| Converter config registry | `tests/framework_configs/converters.py` |
| Output adapters | `tests/framework_configs/output_adapters.py` |
| Shared tool-event fixtures | `tests/framework_configs/fixtures.py` |
| Adapter conformance tests | `tests/framework_conformance/test_adapter_conformance.py` |
| Converter conformance tests | `tests/framework_conformance/test_converter_conformance.py` |
| Config drift detection | `tests/framework_conformance/test_config_drift.py` |
| Framework-specific adapter tests | `tests/adapters/test_$ARGUMENTS_adapter.py` |
| Framework-specific converter tests | `tests/converters/test_$ARGUMENTS.py` |
| Excluded modules (adapters) | `ADAPTER_EXCLUDED_MODULES` in `tests/framework_configs/adapters.py` |
| Excluded modules (converters) | `CONVERTER_EXCLUDED_MODULES` in `tests/framework_configs/converters.py` |
