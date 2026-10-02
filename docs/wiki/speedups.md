# Performance Optimizations

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

MarkupSafe provides an optional C extension module that accelerates the critical HTML escaping function for better performance with large text operations. The speedups are optional: if the C extension fails to build, the pure Python implementation is used transparently.

## Architecture

The escaping function is implemented twice:

- **Pure Python**: [`src/markupsafe/_native.py`](../../src/markupsafe/_native.py) provides the reference implementation using string replacements
- **C extension**: [`src/markupsafe/_speedups.c`](../../src/markupsafe/_speedups.c) provides an optimized version for production use

Both implement the same interface and are swapped in at import time. Users of the [Markup Class](markup-class.md) do not need to know which version is active.

## C Extension Design

The C module [`src/markupsafe/_speedups.c`](../../src/markupsafe/_speedups.c) optimizes HTML escaping by:

1. **Pre-calculating output size**: The `GET_DELTA` macro [`src/markupsafe/_speedups.c:3-16`](../../src/markupsafe/_speedups.c#L3-L16) scans the input string once to determine how much extra space is needed. Characters like `"`, `'`, and `&` expand to 5 characters (e.g., `&amp;`), while `<` and `>` expand to 4 characters.

2. **Early return for unchanged strings**: If no special characters are found (delta = 0), the input string is returned unchanged with an incremented reference count [`src/markupsafe/_speedups.c:84-87`](../../src/markupsafe/_speedups.c#L84-L87).

3. **Single-pass escaping**: The `DO_ESCAPE` macro [`src/markupsafe/_speedups.c:18-72`](../../src/markupsafe/_speedups.c#L18-L72) performs the actual escaping in one pass. It buffers consecutive unescaped characters and copies them in chunks via `memcpy`, then writes the HTML entity sequences inline. This reduces the overhead of character-by-character processing.

4. **Unicode kind dispatch**: Python's flexible Unicode representation uses different internal formats (1-byte, 2-byte, 4-byte) depending on the string content. Three separate functions handle each case [`src/markupsafe/_speedups.c:75-149`](../../src/markupsafe/_speedups.c#L75-L149), avoiding unnecessary conversions. The dispatcher [`src/markupsafe/_speedups.c:151-171`](../../src/markupsafe/_speedups.c#L151-L171) selects the appropriate implementation based on the input string's Unicode kind.

## Escaping Rules

Both implementations escape the same five characters according to HTML rules:

- `&` → `&amp;` (5 chars)
- `<` → `&lt;` (4 chars)
- `>` → `&gt;` (4 chars)
- `"` → `&#34;` (5 chars)
- `'` → `&#39;` (5 chars)

See [Escaping and Safety](escaping.md) for the security rationale.

## GIL Considerations

The C module declares that it does not require the Global Interpreter Lock [`src/markupsafe/_speedups.c:179-184`](../../src/markupsafe/_speedups.c#L179-L184). This is safe because the function only reads the input string and writes to a newly allocated output buffer, with no access to shared state or Python objects during the hot path.

## Building and Distribution

The C extension is built as part of the package [build process](packaging-and-build.md). It is optional: source distributions include both the C code and Python fallback, and wheels may be distributed with or without the compiled extension. The import mechanism tries the C version first and silently falls back to the pure Python version if unavailable.

The module is [tested](testing.md) to ensure both implementations produce identical output across various input strings and edge cases.
