# TABNINE.md

## Project Overview

**Spec Kit** is an open-source toolkit by GitHub for **Spec-Driven Development (SDD)** — a methodology where specifications are created first and then drive implementation via AI coding agents. The project ships:

- **Specify CLI** (`specify`): A Python CLI that bootstraps projects with templates, scripts, and AI agent integrations for the SDD workflow.
- **Templates & Commands**: Markdown/TOML command templates for 18+ AI agents (Claude Code, Gemini, Copilot, Cursor, Windsurf, etc.).
- **Extension System**: A modular plugin architecture for adding commands and functionality without bloating the core.
- **Scripts**: Bash and PowerShell helpers for feature setup, prerequisite checks, and agent context updates.

### Tech Stack

- **Language**: Python 3.11+
- **Package Manager**: [uv](https://docs.astral.sh/uv/) (with `hatchling` build backend)
- **CLI Framework**: Typer + Rich (for terminal UI)
- **HTTP Client**: httpx
- **YAML/Config**: PyYAML, packaging
- **Linting**: Ruff (Python), markdownlint-cli2 (Markdown)
- **Testing**: pytest + pytest-cov

### Architecture

- `src/specify_cli/__init__.py` — Main CLI entry point (~2400 lines). Contains `AGENT_CONFIG` (the single source of truth for all agent metadata), `init` and `check` commands, and all project bootstrapping logic.
- `src/specify_cli/extensions.py` — Extension system (manifest validation, registry, installation/removal, command registration).
- `templates/` — Spec, plan, tasks, and command templates used during project initialization.
- `scripts/bash/` and `scripts/powershell/` — Shell scripts for feature creation, prerequisites, and agent context updates.
- `extensions/` — Extension documentation, catalogs, and the extension template.
- `tests/` — pytest test suite.

## Building and Running

```bash
# Install dependencies
uv sync

# Install test dependencies
uv sync --extra test

# Run the CLI
uv run specify --help
uv run specify init <project-name> --ai claude
uv run specify check

# Run tests
uv run pytest

# Run tests with coverage
uv run pytest --cov=src

# Lint Python
uvx ruff check src/

# Lint Markdown
npx markdownlint-cli2 '**/*.md' '!extensions/**/*.md'

# Build release packages locally (for testing template/command changes)
./.github/workflows/scripts/create-release-packages.sh v1.0.0
```

## Development Conventions

### General Rules (from AGENTS.md)

- Any changes to `__init__.py` for the Specify CLI **require a version bump** in `pyproject.toml` and an entry in `CHANGELOG.md`.
- When adding a new AI agent, the `AGENT_CONFIG` dictionary key **must** match the actual CLI executable name (e.g., `"cursor-agent"` not `"cursor"`). This eliminates special-case mappings.
- New agents require updates across: `AGENT_CONFIG`, CLI help text, `README.md`, release scripts, and agent context update scripts (both bash and PowerShell).

### Code Style

- Python source follows Ruff defaults (no local ruff config — uses defaults).
- Markdown follows markdownlint-cli2 rules configured in `.markdownlint-cli2.jsonc` (ATX headings, 2-space indent, line length unrestricted).
- Use docstrings for modules, classes, and public functions.
- Type hints are used throughout (`typing.Optional`, `typing.Dict`, etc.).

### Testing

- Tests live in `tests/` and follow `test_*.py` naming.
- pytest is configured with `-v`, `--strict-markers`, `--tb=short`.
- Tests run on Python 3.11, 3.12, and 3.13 in CI.
- Use `tempfile.mkdtemp()` for test fixtures requiring filesystem isolation.
- When adding features or fixing bugs, add or update corresponding tests.

### CI Workflows

- **`test.yml`**: Runs Ruff lint on `src/` and pytest across Python 3.11–3.13.
- **`lint.yml`**: Runs markdownlint-cli2 on all Markdown files (excluding `extensions/`).
- **`codeql.yml`**: CodeQL security analysis.
- **`release.yml`**: Automated release pipeline.

### Key Files

| File | Purpose |
|---|---|
| `pyproject.toml` | Project metadata, dependencies, pytest config |
| `src/specify_cli/__init__.py` | Main CLI (all commands, agent config) |
| `src/specify_cli/extensions.py` | Extension system implementation |
| `AGENTS.md` | Detailed guide for adding new AI agent support |
| `CONTRIBUTING.md` | Contribution guidelines and development workflow |
| `CHANGELOG.md` | Version history (update on every CLI change) |
| `templates/` | Spec/plan/task/command templates |
| `scripts/` | Bash and PowerShell helper scripts |
