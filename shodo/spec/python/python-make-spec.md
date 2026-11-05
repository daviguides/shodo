# Python Make Targets

Contract: `spec/make-contract-spec.md`. Commands use `uv run` so the
locked environment is used and nothing is installed.

| Target | Commands |
|--------|----------|
| `test` | `uv run pytest` |
| `check` | `uv run ruff check <package> tests`; `uv run ruff format --check <package> tests`; `uv run mypy --strict <package>` only when the project already type-checks with mypy (mypy in dev dependencies or `[tool.mypy]` in `pyproject.toml`) |

```make
.PHONY: install test check format

install:
	uv sync

test:
	uv run pytest

check:
	uv run ruff check mypkg tests
	uv run ruff format --check mypkg tests
	uv run mypy --strict mypkg

format:
	uv run ruff format mypkg tests
	uv run ruff check --fix mypkg tests
```

`<package>` is the import package of the component (the directory
under the component root or `src/`). Coverage flags (`--cov`,
`--cov-branch`) live in `pyproject.toml` `addopts`, not in the recipe;
no `--cov-fail-under` gate (kinhin: coverage after prune is a smoke
alarm, not a gate).
