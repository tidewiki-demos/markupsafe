# Markup Class

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

The `Markup` class represents HTML-safe text that is ready to be inserted into HTML or XML documents. It is a subclass of `str` that automatically escapes untrusted input in string operations while preserving text that has already been marked safe.

## Core Concept

[`src/markupsafe/__init__.py:84-118`](../../src/markupsafe/__init__.py#L84-L118) The `Markup` class wraps strings to indicate they are safe for HTML insertion. When you pass an object to the constructor, it converts it to text and marks it safe without escaping. To escape text before wrapping it, use the `escape()` class method instead. The class implements the `__html__()` interface: if an object passed to the constructor has an `__html__()` method, that method is called and its result is wrapped as safe.

## Safety Through Escaping

All string operations on `Markup` instances automatically escape their arguments before combining them with the safe text. This ensures that untrusted input cannot inject unescaped HTML. For example, [`src/markupsafe/__init__.py:114-117`](../../src/markupsafe/__init__.py#L114-L117) using percent formatting or concatenation with unsafe strings escapes those strings in the result.

The [`src/markupsafe/__init__.py:136-140`](../../src/markupsafe/__init__.py#L136-L140) `__add__` method and [`src/markupsafe/__init__.py:142-146`](../../src/markupsafe/__init__.py#L142-L146) `__radd__` method both call `self.escape()` on the value being added. Similarly, [`src/markupsafe/__init__.py:257-258`](../../src/markupsafe/__init__.py#L257-L258) the `replace()` method escapes the replacement string, and [`src/markupsafe/__init__.py:260-273`](../../src/markupsafe/__init__.py#L260-L273) padding methods like `ljust()`, `rjust()`, and `center()` escape their fill characters.

## String Operations

Most string methods are overridden to return `Markup` instances instead of plain strings:

- [`src/markupsafe/__init__.py:242-243`](../../src/markupsafe/__init__.py#L242-L243) Slicing via `__getitem__` returns a `Markup` instance
- [`src/markupsafe/__init__.py:173-186`](../../src/markupsafe/__init__.py#L173-L186) Splitting methods (`split()`, `rsplit()`, `splitlines()`) return lists of `Markup` instances
- [`src/markupsafe/__init__.py:245-295`](../../src/markupsafe/__init__.py#L245-L295) Case and whitespace methods (`capitalize()`, `lower()`, `upper()`, `strip()`, `lstrip()`, `rstrip()`, etc.) all return `Markup` instances
- [`src/markupsafe/__init__.py:303-311`](../../src/markupsafe/__init__.py#L303-L311) Partition methods (`partition()`, `rpartition()`) return tuples of `Markup` instances

## Formatting

The `Markup` class provides two formatting mechanisms:

### format() Method

[`src/markupsafe/__init__.py:313-323`](../../src/markupsafe/__init__.py#L313-L323) The `format()` and `format_map()` methods use an `EscapeFormatter` to handle format strings. This formatter checks for three things in order: [`src/markupsafe/__init__.py:340-354`](../../src/markupsafe/__init__.py#L340-L354)

1. If the value has an `__html_format__()` method, it is called with the format specifier
2. If the value has an `__html__()` method (but not `__html_format__()`), it is called only if no format specifier was given
3. Otherwise, Python's default format behavior is used and the result is escaped

A `ValueError` is raised if a format specifier is provided but the value only implements `__html__()` and not `__html_format__()`.

See [Formatting](docs/formatting.rst) for examples, including how to implement custom classes with `__html_format__()` methods.

### Percent (printf-style) Formatting

[`src/markupsafe/__init__.py:154-165`](../../src/markupsafe/__init__.py#L154-L165) The `__mod__` method handles percent formatting. Arguments are wrapped in a `_MarkupEscapeHelper` that escapes them when they are formatted into the string. Tuples of arguments are each wrapped; mappings are wrapped as a whole; single arguments are wrapped with a tuple wrapper.

## HTML Interface

[`src/markupsafe/__init__.py:133-134`](../../src/markupsafe/__init__.py#L133-L134) The `__html__()` method returns itself, signaling that the `Markup` instance is safe for HTML insertion. [`src/markupsafe/__init__.py:325-329`](../../src/markupsafe/__init__.py#L325-L329) The `__html_format__()` method rejects format specifications (raising a `ValueError` if one is provided) and returns the instance itself.

## Utility Methods

[`src/markupsafe/__init__.py:188-197`](../../src/markupsafe/__init__.py#L188-L197) The `unescape()` method converts escaped markup back to text by replacing HTML entities with their corresponding characters.

[`src/markupsafe/__init__.py:199-228`](../../src/markupsafe/__init__.py#L199-L228) The `striptags()` method removes HTML tags and comments from the markup, unescapes entities, and normalizes whitespace to single spaces. It handles comments and tags separately to avoid premature termination when a comment contains a tag.

[`src/markupsafe/__init__.py:230-240`](../../src/markupsafe/__init__.py#L230-L240) The `escape()` class method wraps the global `escape()` function and ensures subclasses receive the correct type.

## Related Functions

The module also provides helper functions for escaping:

- [`src/markupsafe/__init__.py:24-45`](../../src/markupsafe/__init__.py#L24-L45) `escape()` replaces dangerous characters (`&`, `<`, `>`, `'`, `"`) with HTML-safe sequences. It checks for `__html__()` methods and handles the object-to-string conversion efficiently.
- [`src/markupsafe/__init__.py:48-61`](../../src/markupsafe/__init__.py#L48-L61) `escape_silent()` is like `escape()` but treats `None` as an empty string instead of the literal string `'None'`.
- [`src/markupsafe/__init__.py:64-81`](../../src/markupsafe/__init__.py#L64-L81) `soft_str()` converts an object to a string while preserving `Markup` instances, preventing double-escaping.

See [Escaping and Safety](escaping.md) for more details on these functions and the escaping strategy.
