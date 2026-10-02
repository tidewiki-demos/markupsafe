# CI and Release Workflow

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

MarkupSafe uses GitHub Actions workflows to automate testing, code quality checks, and publishing to PyPI. The workflows run on pull requests and pushes to main branches, with a separate process for creating releases and publishing packages.

## Testing Workflow

[`.github/workflows/tests.yaml:1-55`](../../.github/workflows/tests.yaml#L1-L55)

The `Tests` workflow runs on pull requests and pushes to `main` and `stable` branches (excluding documentation changes). It uses a matrix strategy to test across multiple Python versions and platforms:

- Python 3.10 through 3.14, including pre-release versions
- Python 3.13t and 3.14t (free-threaded builds)
- PyPy 3.11
- Windows, macOS, and Linux operating systems

The workflow runs `tox` with `uv` as the package manager to execute test environments. For free-threaded Python builds (identified by the `t` suffix), an additional `parallel` test environment runs to validate thread safety. A separate `typing` job runs mypy for static type checking with caching.

See [Testing and Quality Assurance](testing.md) for details on the test suite itself.

## Code Quality Checks

[`.github/workflows/pre-commit.yaml:1-25`](../../.github/workflows/pre-commit.yaml#L1-L25)

The `pre-commit` workflow runs on pull requests and pushes to `main` and `stable` branches. It uses the pre-commit framework to enforce code style, formatting, and linting rules defined in `.pre-commit-config.yaml`. The workflow also uses the pre-commit CI lite action for additional integration checks.

## Publishing Workflow

[`.github/workflows/publish.yaml:1-110`](../../.github/workflows/publish.yaml#L1-L110)

The `Publish` workflow builds and releases MarkupSafe to PyPI. It triggers in two ways:

1. **Automatic on tag push**: When a git tag is pushed (e.g., `v2.1.0`), the workflow builds source distributions (sdist) and wheels, creates a draft GitHub release, and prepares for PyPI upload.

2. **Manual via workflow_dispatch**: When triggered manually with inputs for `tag` and `python` version, it rebuilds wheels for a new Python version on an existing tag and updates the draft release.

### Build Steps

**Source Distribution** (`sdist` job):
- Checks out the tagged commit
- Builds the source distribution using `uv build --sdist`
- Uploads the artifact
- Skipped on manual `workflow_dispatch` runs (new Python version builds don't need a new sdist)

**Wheels** (`wheels` job):
- Runs on Ubuntu, Windows, and macOS
- Sets up QEMU on Linux to build arm64 and riscv64 architectures
- Uses `cibuildwheel` to build wheels for all supported Python versions
- For `workflow_dispatch` runs, only builds wheels for the specified Python version using the `CIBW_BUILD` environment variable

Both jobs set `SOURCE_DATE_EPOCH` for reproducible builds.

### Release and Upload

**Create Release** (`create-release` job):
- On tag push: Creates a new draft release on GitHub with all built artifacts
- On manual dispatch: Uploads new artifacts to the existing release

**Publish to PyPI** (`publish-pypi` job):
- Requires approval via GitHub's `publish` environment before uploading
- Uses trusted publishing (OpenID Connect) for secure authentication
- Skips uploading if files already exist on PyPI (via `skip-existing` flag)

## Issue and PR Locking

[`.github/workflows/lock.yaml:1-24`](../../.github/workflows/lock.yaml#L1-L24)

The `Lock inactive closed issues` workflow runs daily. It automatically locks issues, pull requests, and discussions that have been closed and inactive for 14 days. This reduces noise in old issues while allowing fresh discussions on newer ones.

## Decisions

**Two-stage release process**: The `publish.yaml` workflow creates a draft release for human review before uploading to PyPI. This allows maintainers to inspect built artifacts and verify correctness before making them publicly available. The workflow requires explicit approval through GitHub's environment protection rules ([`.github/workflows/publish.yaml:92-101`](../../.github/workflows/publish.yaml#L92-L101)).

**Manual Python version wheel building**: The `workflow_dispatch` input allows rebuilding wheels for a new Python version without requiring a new source distribution or recreating the git tag. This supports rapid responses when Python releases are published ([`.github/workflows/publish.yaml:5-14`](../../.github/workflows/publish.yaml#L5-L14)).

**Reproducible builds**: Both `sdist` and `wheels` jobs set `SOURCE_DATE_EPOCH` to the commit timestamp for reproducible builds, ensuring builds are deterministic across different machines and times ([[cite:.github/workflows/publish.yaml:29,56]]).
