# Development Environment

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

This page describes the tools and configuration for setting up a local development environment for MarkupSafe, and the code quality standards enforced across the project.

## Quick Start with Dev Containers

The repository includes a [Dev Container](https://containers.dev/) configuration for a consistent development experience. To use it:

1. Open the repository in a Dev Container-compatible editor (such as VS Code with the Dev Containers extension).
2. The container is built from [`.devcontainer/devcontainer.json:3`](../../.devcontainer/devcontainer.json#L3) the official Python dev container image.
3. On creation, [`.devcontainer/on-create-command.sh:16-17`](../../.devcontainer/on-create-command.sh#L16-L17) the setup script installs pre-commit hooks and creates a virtual environment.

The Dev Container automatically configures the Python interpreter path [`.devcontainer/devcontainer.json:7`](../../.devcontainer/devcontainer.json#L7) and activates the virtual environment in terminals [`.devcontainer/devcontainer.json:8`](../../.devcontainer/devcontainer.json#L8).

## Manual Local Setup

If you do not use a Dev Container:

1. **Install uv**: The project uses `uv` for dependency management. Install it from https://astral.sh/uv or use your system package manager.
2. **Create virtual environment and install dependencies**: Run `uv sync` to create a `.venv` directory and install all project dependencies.
3. **Install pre-commit hooks**: Run `pre-commit install --install-hooks` to enable automatic code quality checks before commits.

## Code Quality Tools

The project enforces code quality through automated tools configured in [`.pre-commit-config.yaml`](../../.pre-commit-config.yaml):

- **Ruff** [`.pre-commit-config.yaml:2-6`](../../.pre-commit-config.yaml#L2-L6): A fast Python linter and formatter. It checks code style and format, and can automatically fix issues with `ruff format`.
- **uv lock check** [`.pre-commit-config.yaml:7-10`](../../.pre-commit-config.yaml#L7-L10): Ensures the `uv.lock` file is up-to-date with `pyproject.toml` changes.
- **Pre-commit hooks** [`.pre-commit-config.yaml:11-18`](../../.pre-commit-config.yaml#L11-L18): General checks including merge conflict markers, debug statements, byte-order markers, trailing whitespace, and EOF formatting.

These tools run automatically on staged files before each commit. To run them manually on all files, use `pre-commit run --all-files`.

## Code Style

The project follows these formatting conventions, defined in [`.editorconfig`](../../.editorconfig):

- **Indentation**: 4 spaces for Python files; 2 spaces for web assets (CSS, HTML, JavaScript, JSON, YAML).
- **Line ending**: LF (Unix-style).
- **Character encoding**: UTF-8.
- **Line length**: Maximum 88 characters [`.editorconfig:10`](../../.editorconfig#L10), enforced by Ruff's formatter.
- **Whitespace**: No trailing whitespace; final newline required on all files.

Use an EditorConfig-compatible editor to apply these settings automatically.

## Before Opening a Pull Request

Before submitting code, ensure that:

1. **Tests pass**: Refer to [Testing and Quality Assurance](testing.md) for running tests locally.
2. **Pre-commit checks pass**: Commit hooks will verify code quality automatically. If they fail, fix the issues and stage the corrected files again.
3. **Documentation is updated**: Add or update docstrings, docs files, and changelog entries. See [Documentation](documentation.md) for details.
4. **An issue exists**: [`.github/pull_request_template.md:1-4`](../../.github/pull_request_template.md#L1-L4) Open a GitHub issue first describing the bug or feature (not required for typos or simple non-code changes).

See [`.github/pull_request_template.md`](../../.github/pull_request_template.md) for the full pull request checklist.

## Issue Templates

The project provides templates for [bug reports](https://github.com/pallets/markupsafe/issues/new?template=bug-report.md) and [feature requests](https://github.com/pallets/markupsafe/issues/new?template=feature-request.md). [`.github/ISSUE_TEMPLATE/config.yml`](../../.github/ISSUE_TEMPLATE/config.yml) directs general questions to GitHub Discussions or the Pallets Discord instead of the issue tracker.

## Additional Resources

- [Testing and Quality Assurance](testing.md) — Running tests and quality checks.
- [Packaging and Build](packaging-and-build.md) — Building distributions.
- [CI and Release Workflow](ci-and-release.md) — Automated checks and release process.
