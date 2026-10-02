# Documentation

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

MarkupSafe uses Sphinx to generate user guides and API documentation, published on Read the Docs. The documentation covers escaping, HTML representations, string formatting, and API references for the [Markup Class](markup-class.md).

## Build System

Documentation is built using Sphinx and managed through two main entry points:

- **Local development**: The `docs/Makefile` (Unix) and `docs/make.bat` (Windows) provide a standard Sphinx build interface. Running `make html` in the `docs/` directory generates HTML output to `_build/html/`.
- **Read the Docs**: The `.readthedocs.yaml` file configures automated builds on pull requests and releases [`.readthedocs.yaml`](../../.readthedocs.yaml). It installs the `uv` package manager via asdf, then runs `sphinx-build -W -b dirhtml` with the `docs` group dependencies. The `-W` flag treats warnings as errors, ensuring documentation quality. Output is placed in `$READTHEDOCS_OUTPUT/html` using the `dirhtml` builder, which creates a directory-based HTML structure [`.readthedocs.yaml:10`](../../.readthedocs.yaml#L10).

## Configuration

[`docs/conf.py`](../../docs/conf.py) defines the Sphinx configuration:

- **Extensions**: Sphinx's built-in `autodoc` generates API documentation from docstrings, `extlinks` provides shorthand links to GitHub issues and PRs, `intersphinx` links to Python's standard documentation, `sphinxcontrib.log_cabinet` manages changelog formatting, and `pallets_sphinx_themes` supplies the Jinja theme [`docs/conf.py:14-20`](../../docs/conf.py#L14-L20).
- **Autodoc settings**: Member order follows source order, type hints appear in descriptions, and default values are preserved [`docs/conf.py:21-23`](../../docs/conf.py#L21-L23).
- **Theme**: Uses the "jinja" theme from `pallets_sphinx_themes` with sidebar configuration showing project links (Donate, PyPI Releases, Source Code, Issue Tracker, Chat) and search functionality [`docs/conf.py:34-48`](../../docs/conf.py#L34-L48).
- **Project metadata**: Title, version, and copyright are auto-detected from the package; the favicon and logo reference SVG assets [[cite:docs/conf.py:6-9,51-53]].

## Content Structure

The documentation is organized around key concepts:

- **escaping.rst**: Documents the `escape` function, `Markup` class methods (`escape`, `unescape`, `striptags`), and utility functions for optional values and string conversion [`docs/escaping.rst`](../../docs/escaping.rst). See [Escaping and Safety](escaping.md) for implementation details.
- **html.rst**: Explains the `__html__` protocol, showing how objects can define custom HTML representations that bypass escaping [`docs/html.rst`](../../docs/html.rst).
- **formatting.rst**: Describes how `Markup` handles string formatting with the `__html_format__` method, providing control over escaping in format strings [`docs/formatting.rst`](../../docs/formatting.rst).
- **changes.rst**: Includes the changelog from `CHANGES.rst` [`docs/changes.rst`](../../docs/changes.rst).
- **license.rst**: Displays the BSD-3-Clause license from `LICENSE.txt` [`docs/license.rst`](../../docs/license.rst).
- **index.rst**: Landing page with overview, installation instructions, and table of contents [`docs/index.rst`](../../docs/index.rst).

## Building Documentation Locally

Use the Makefile:

```bash
cd docs
make html
```

Output appears in `docs/_build/html/`. To rebuild cleanly, run `make clean` first. The default role is set to `code` [`docs/conf.py:13`](../../docs/conf.py#L13), so backticks render as inline code without explicit role markup.
