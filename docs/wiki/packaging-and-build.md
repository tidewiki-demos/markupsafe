# Packaging and Build

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

This page covers the build configuration, dependencies, and distribution setup for MarkupSafe as a Python library. MarkupSafe is distributed with optional native C extension support for performance, but can fall back to a pure Python implementation.

## Project Metadata

[`pyproject.toml:1-27`](../../pyproject.toml#L1-L27) defines the core project metadata. MarkupSafe is maintained by Pallets and licensed under the BSD-3-Clause license [`LICENSE.txt`](../../LICENSE.txt). The project requires Python 3.10 or later [`pyproject.toml:19`](../../pyproject.toml#L19).

## Build System

[`pyproject.toml:59-61`](../../pyproject.toml#L59-L61) specifies setuptools as the build backend, requiring version 77 or later. The project uses a two-stage setup: setuptools handles the build, and a custom `setup.py` manages native extension compilation.

## Native Extension Support

MarkupSafe includes an optional C extension for performance-critical operations. [`setup.py:12`](../../setup.py#L12) defines the `markupsafe._speedups` extension module, which compiles from [`setup.py:12`](../../setup.py#L12) `src/markupsafe/_speedups.c`. See [Performance Optimizations](speedups.md) for details on what the extension implements.

### Graceful Fallback Mechanism

The build is designed to succeed even if the C extension cannot be compiled. [`setup.py:19-37`](../../setup.py#L19-L37) defines a custom `ve_build_ext` class that catches compilation errors and converts them to `BuildFailed` exceptions.

[`setup.py:54-77`](../../setup.py#L54-L77) implements the fallback logic:

- On PyPy, Jython, and GraalVM, the pure Python implementation is always used [`setup.py:54-58`](../../setup.py#L54-L58)
- Under `cibuildwheel`, compilation is required for the platform [`setup.py:60-61`](../../setup.py#L60-L61)
- In normal development, the build attempts compilation but falls back to pure Python if it fails [`setup.py:62-75`](../../setup.py#L62-L75)
- Users see a warning message when the extension is unavailable [[cite:setup.py:47-51,66-82]]

The pure Python fallback is [`src/markupsafe/_native.py`](../../src/markupsafe/_native.py), and the module imports from it if the C extension is not available [`src/markupsafe/__init__.py:7-10`](../../src/markupsafe/__init__.py#L7-L10).

## Distribution Configuration

### Package Contents

[`MANIFEST.in`](../../MANIFEST.in) specifies what files are included in source distributions:

- Documentation and test files are included
- Type hints and `.pyi` stub files are distributed [`MANIFEST.in:6-7`](../../MANIFEST.in#L6-L7)
- Compiled Python bytecode is excluded [`MANIFEST.in:8`](../../MANIFEST.in#L8)

### Wheel Building

[`pyproject.toml:206-222`](../../pyproject.toml#L206-L222) configures `cibuildwheel` for multi-platform wheel builds:

- CPython free-threading support is enabled [`pyproject.toml:207`](../../pyproject.toml#L207)
- The build frontend is configured to use `build[uv]` for faster builds [`pyproject.toml:208`](../../pyproject.toml#L208), except on `musllinux_riscv64` where it falls back to `build` [`pyproject.toml:210-213`](../../pyproject.toml#L210-L213)
- Linux builds target x86_64, aarch64, and riscv64 [`pyproject.toml:215-216`](../../pyproject.toml#L215-L216)
- macOS builds target x86_64 and arm64 (Apple Silicon) [`pyproject.toml:218-219`](../../pyproject.toml#L218-L219)
- Windows builds target auto and ARM64 [`pyproject.toml:221-222`](../../pyproject.toml#L221-L222)

## Dependency Groups

[`pyproject.toml:28-57`](../../pyproject.toml#L28-L57) defines optional dependency groups managed with `uv`:

- **dev**: Linting (ruff), testing framework (tox), and uv integration for tox
- **tests**: pytest and parallel test runner for Python 3.13+
- **typing**: Static type checkers (mypy, pyright) and pytest for type checking tests
- **docs**: Sphinx and Pallets themes for documentation builds
- **pre-commit**: Pre-commit hooks framework and uv integration

[`pyproject.toml:64`](../../pyproject.toml#L64) sets the default groups for development environments, and the lockfile is maintained in `uv.lock` [`MANIFEST.in:2`](../../MANIFEST.in#L2).

## Type Hints

The package is fully typed. [`src/markupsafe/_speedups.pyi`](../../src/markupsafe/_speedups.pyi) provides type stubs for the C extension. See [Testing and Quality Assurance](testing.md) for information on type checking configuration.
