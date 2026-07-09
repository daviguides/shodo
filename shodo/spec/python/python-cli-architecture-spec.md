# Python CLI Architecture Specification

## Package Structure

Package name in `snake_case` matching the project name (hyphenated → underscore).

```
project-name/
├── pyproject.toml
├── project_name/
│   ├── __init__.py          # Package + __version__
│   ├── main.py              # Aggregator — registers commands only
│   ├── display.py           # Rich console + semantic output helpers
│   │
│   ├── commands/            # Interface layer
│   ├── services/            # Business logic layer
│   ├── schemas/             # Pydantic schemas (data representation)
│   ├── integrations/        # Low-level I/O (filesystem, subprocess, APIs)
│   └── data/                # Bundled config (YAML, shipped with package)
│
└── tests/
    └── project_name/
```

## Layered Architecture

Five layers with strict dependency direction.

```
commands/ → services/ → schemas/
                      → integrations/
                      → data/
```

### Layer Definitions

**`main.py`** — Pure aggregator. Creates the root `typer.Typer()` and registers subcommand groups via `app.add_typer()`. Zero logic. Under 40 lines.

**`commands/`** — Interface layer. Each module creates its own `typer.Typer()`, defines commands with `@app.command()`. Handles argument parsing, output formatting, error display. Calls `services/` for business logic. Never executes domain logic directly.

**`services/`** — Business logic. Pure functions and classes operating on domain concepts. No typer imports. No terminal output. Receives typed data, returns typed data. Reusable across commands.

**`schemas/`** — Pydantic models defining data structures. Pure representation — no logic, no I/O. Validates data shape and types.

**`integrations/`** — Low-level operations. Filesystem, subprocess wrappers (git, gh), YAML I/O, API clients. Domain-agnostic — knows nothing about business concepts.

**`data/`** — Bundled config files shipped with the package. Accessed via `Path(__file__)` relative resolution. Not user-editable at runtime.

**`display.py`** — Shared `Console` singletons and semantic output functions (`print_success`, `print_error`, `print_warning`). Imported by `commands/` only.

## Dependency Rules

| Layer | Can import from | Cannot import from |
|-------|----------------|-------------------|
| `commands/` | `services/`, `schemas/`, `display.py` | — |
| `services/` | `schemas/`, `integrations/`, `data/` | `commands/`, `display.py` |
| `schemas/` | stdlib, pydantic | anything else |
| `integrations/` | stdlib, third-party | `commands/`, `services/`, `schemas/` |
| `display.py` | `rich` | `commands/`, `services/` |

## Responsibility Split Example

A feature that loads config from YAML and validates it:

| Responsibility | Layer | What it knows |
|---------------|-------|---------------|
| Generic YAML read/write | `integrations/` | How to parse YAML files |
| Where config lives, how to resolve and validate | `services/` | Domain rules, file locations |
| Data structure definition | `schemas/` | Field names, types, constraints |
| User-facing args and output | `commands/` | CLI flags, rich formatting |

## Sub-App Architecture (Context Separation)

The flat layout above is the starting point. When command groups
develop distinct domains — services clustering by theme, schemas used
by only one group — graduate to **sub-apps**: vertical slices that
replicate the layers internally.

```
project_name/
├── main.py              # Aggregator — registers core + sub-apps
├── display.py           # Shared across all sub-apps
├── exceptions.py        # Shared error hierarchy
├── data/                # Bundled config
│
├── core/                # Sub-app for SHARED domain
│   ├── schemas/
│   ├── integrations/
│   ├── services/
│   └── commands/
│
├── commit/              # Sub-app = vertical slice
│   ├── schemas/
│   ├── integrations/
│   ├── services/
│   └── commands/
├── pull/                # Smaller slice — only the layers it needs
│   ├── services/
│   └── commands/
└── sync/
    ├── services/
    └── commands/
```

### Sub-App Rules

1. **Each sub-app replicates only the layers it needs** — a slice
   with two layers is correct, not incomplete. Dependency rules apply
   WITHIN the slice.
2. **`core/` owns the shared domain** — sub-apps import from `core/`,
   **never from each other**. Logic needed by two slices moves to core.
3. **Top-level shared**: `main.py`, `display.py`, `exceptions.py`,
   `data/` — transversal to all slices.
4. **`main.py` stays a pure aggregator** — each sub-app exposes its
   own `typer.Typer()` from `<subapp>/commands/`, registered via
   `app.add_typer(commit_app, name="commit")`.

### When to Graduate

| Signal | Action |
|--------|--------|
| Command groups share no services | Split into sub-apps |
| `services/` clusters by theme | Each cluster becomes a slice |
| A schema is used by one group only | It belongs inside that slice |
| Two slices need the same service | Move it to `core/` |

The directory listing should scream what the tool DOES (commit, pull,
sync), not how it is built (commands, services).

## API Analogy

| API Concept | CLI Equivalent |
|-------------|---------------|
| FastAPI instance | `main.py` (Typer instance) |
| `routes/` | `commands/` |
| `services/` | `services/` |
| `schemas/` | `schemas/` |
| `integrations/` | `integrations/` |
| `static/` | `data/` |
| middleware/logging | `display.py` |

## Entrypoint Convention

```toml
[project.scripts]
cli-name = "package_name.main:app"
```

`main.py` is the single entrypoint. It composes the CLI from subcommand modules:

```python
"""CLI entrypoint. Registers subcommand groups."""

import typer

from package_name.commands.sync import app as sync_app

app = typer.Typer(
    name="cli-name",
    help="CLI description.",
    no_args_is_help=True,
)

app.add_typer(sync_app, name="sync")
```

## pyproject.toml Convention

```toml
[project]
name = "project-name"
version = "0.1.0"
description = "..."
requires-python = ">=3.14"
dependencies = [
    "typer>=0.15.0",
    "rich>=13.9.0",
    "pyyaml>=6.0.0",
    "pydantic>=2.0.0",
]

[project.scripts]
cli-name = "package_name.main:app"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.ruff]
line-length = 80
target-version = "py314"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]

[dependency-groups]
dev = [
    "pytest>=8.0.0",
    "ruff>=0.15.0",
]

[tool.pytest.ini_options]
testpaths = ["tests"]
```
