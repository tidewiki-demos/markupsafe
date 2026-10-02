# Escaping and Safety

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

MarkupSafe provides functions and mechanisms to escape special characters and prevent injection attacks in HTML and other markup contexts. The core strategy is to encode dangerous characters as HTML entities while preserving safe content.

## Core Escape Function

The [`tests/test_escape.py:7`](../../tests/test_escape.py#L7) `escape()` function converts unsafe strings into safe markup by encoding special characters. It replaces:
- `&` with `&amp;`
- `>` with `&gt;`
- `<` with `&lt;`
- `'` (single quote) with `&#39;`
- `"` (double quote) with `&#34;`

[`tests/test_escape.py:11-34`](../../tests/test_escape.py#L11-L34) The function handles various input types including:
- Empty strings
- ASCII text with special characters
- Multi-byte Unicode characters (2-byte and 4-byte sequences)
- Mixed content with safe and unsafe characters

The function returns a [`tests/test_escape.py:34`](../../tests/test_escape.py#L34) `Markup` object, which marks the result as safe and prevents re-escaping.

## Handling Edge Cases

The escape function robustly handles non-standard string-like objects:

[`tests/test_escape.py:37-55`](../../tests/test_escape.py#L37-L55) A proxy object that masquerades as a string (with a `__class__` property that returns `str`) is correctly detected and escaped as plain text rather than treated as already-safe markup.

[`tests/test_escape.py:58-68`](../../tests/test_escape.py#L58-L68) String subclasses where `__str__()` returns the subclass itself (rather than a plain `str`) are handled correctly without infinite recursion or type errors.

## Custom HTML Protocol

Objects can define a `__html__()` method to provide custom HTML representations. [`tests/test_exception_custom_html.py:8-23`](../../tests/test_exception_custom_html.py#L8-L23) If a `__html__()` implementation raises an exception, the exception is propagated correctly to the caller rather than being silently caught or converted to escaped output. This ensures errors in custom markup generation are visible to developers.

## Related Concepts

The escape function works in conjunction with the [Markup Class](markup-class.md), which marks strings as safe from further escaping. For performance-sensitive code, see [Performance Optimizations](speedups.md) for strategies around escaping performance. Comprehensive tests for escaping behavior are documented in [Testing and Quality Assurance](testing.md).
