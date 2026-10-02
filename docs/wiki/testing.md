# Testing and Quality Assurance

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

The test suite verifies correctness and safety across escaping behavior, memory leaks, the Markup class functionality, and exception handling. Tests run against both the native Python implementation and optimized C speedups to ensure consistent behavior.

## Test Structure

The test suite is organized as follows:

- **Escaping behavior** ([test_escape.py](test_escape.py)) verifies that HTML entities are correctly escaped across ASCII, 2-byte, and 4-byte Unicode characters, and handles edge cases like proxy objects and str subclasses.
- **Markup functionality** ([test_markupsafe.py](test_markupsafe.py)) tests the Markup class operations: string interpolation, HTML interoperability via `__html__()`, formatting, and type behavior.
- **Exception handling** ([test_exception_custom_html.py](test_exception_custom_html.py)) ensures exceptions raised in custom `__html__()` implementations are propagated correctly.
- **Memory leaks** ([test_leak.py](test_leak.py)) detects memory leaks during repeated escape operations.
- **Extension module initialization** ([test_ext_init.py](test_ext_init.py)) verifies multi-phase initialization of the speedups module.

## Implementation Details

### Dual Implementation Testing

[`tests/conftest.py:26-39`](../../tests/conftest.py#L26-L39) The test fixture `_mod` parametrizes tests to run against both implementations. Each test automatically runs twice: once using the native Python `_escape_inner` and once using the optimized C `_speedups._escape_inner`. If speedups are unavailable (not compiled), that variant is skipped.

This dual approach ensures that both implementations produce identical results and behave identically across all test cases.

### Escaping Test Coverage

[`tests/test_escape.py:11-34`](../../tests/test_escape.py#L11-L34) The parametrized test `test_escape` covers:
- Empty strings
- ASCII characters including special HTML entities (`&`, `>`, `<`, `'`, `"`)
- 2-byte Unicode (Japanese characters)
- 4-byte Unicode (emoji)

Each case is tested with special characters at the beginning, middle, and end of the string.

[`tests/test_escape.py:37-55`](../../tests/test_escape.py#L37-L55) The `test_proxy` test handles proxy objects that override `__class__` to pretend they are strings, ensuring the escaping logic correctly identifies the actual type.

[`tests/test_escape.py:58-68`](../../tests/test_escape.py#L58-L68) The `test_subclass` test verifies handling of str subclasses where `str(obj)` returns the subclass itself rather than a plain str.

### Markup Class Testing

[`tests/test_markupsafe.py:19-33`](../../tests/test_markupsafe.py#L19-L33) String interpolation tests verify that both positional (`%s`) and keyword (`%(name)s`) formatting properly escape untrusted data while preserving Markup instances.

[`tests/test_markupsafe.py:42-52`](../../tests/test_markupsafe.py#L42-L52) The `test_html_interop` test ensures that objects implementing `__html__()` are recognized and their HTML is preserved without escaping, while their string representation is escaped when used in other contexts.

[`tests/test_markupsafe.py:107-115`](../../tests/test_markupsafe.py#L107-L115) The `format()` and `format_map()` tests verify that format specifications also respect the Markup contract: escaping untrusted data and preserving Markup instances.

### Exception Handling

[`tests/test_exception_custom_html.py:8-23`](../../tests/test_exception_custom_html.py#L8-L23) The exception test creates a class with an `__html__()` method that raises `ValueError`, confirming the exception propagates correctly rather than being silently caught or returning a default value. This addresses a bug previously found in the native implementation (issue #108).

### Memory Leak Detection

[`tests/test_leak.py:10-27`](../../tests/test_leak.py#L10-L27) The leak test repeatedly calls `escape()` in a loop and tracks the count of objects in the garbage collector using `gc.get_objects()`. A true memory leak would cause the count to increase on every iteration; the test allows for up to 2 distinct counts (accounting for JIT stabilization in PyPy and Python 3.13) but fails if counts keep increasing.

## Performance Benchmarking

[`bench.py`](../../bench.py) The `bench.py` script measures escape performance using `pyperf` across different scenarios:

- Short and long strings with HTML to escape
- Short and long plain strings without special characters
- Long strings with a small escaped prefix and a large plain suffix

Each scenario is benchmarked for both the native and speedups implementations, enabling comparison of the C optimization gains. See [Performance Optimizations](speedups.md) for results.

## Thread Safety

Tests marked with `@pytest.mark.thread_unsafe` manipulate global state (such as `sys.modules` in [`tests/test_ext_init.py:13-27`](../../tests/test_ext_init.py#L13-L27) or call `gc.get_objects()` in [`tests/test_leak.py:10`](../../tests/test_leak.py#L10)) and cannot run concurrently with other tests.

## Decisions

- **Dual implementation testing**: Tests run against both native and speedups implementations to catch divergence between them, ensuring users get consistent behavior regardless of which implementation is active. ([`tests/conftest.py`](../../tests/conftest.py))
- **Comprehensive Unicode coverage**: Escape tests include ASCII, 2-byte, and 4-byte Unicode to catch platform-specific or encoding-specific bugs that might only appear with certain character widths.
- **Proxy and subclass edge cases**: The test suite explicitly handles unusual Python objects (proxy objects, str subclasses with unexpected `__str__()` behavior) to ensure robustness.
