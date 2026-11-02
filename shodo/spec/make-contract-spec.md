# Component Makefile Contract

Every component a task touches exposes two Make targets in its own
`Makefile`. A component is the directory holding its manifest
(`pyproject.toml`, `Cargo.toml`, `package.json`, `pom.xml`,
`build.gradle*`, `go.mod`, `.claude-plugin/plugin.json`).

The runner runs `make -C <component> test` and `make -C <component>
check` between IMPLEMENTING and VALIDATING. The language specs
(`spec/<lang>/<lang>-make-spec.md`) give the canonical commands.

## Rules

1. **Location**: the `Makefile` lives in the component directory.
   NEVER add or edit a repo-root `Makefile` to satisfy a component,
   unless the component IS the repo root.
2. **Targets**: `test` and `check` are MANDATORY. Declare both in
   `.PHONY`. Other targets (`install`, `format`) are optional.
3. **`test`**: runs the component's test suite. Exit 0 only when every
   test passes.
4. **`check`**: runs static checks — lint, type check, format check.
   Exit 0 only when all pass. `check` NEVER rewrites files.
5. **Working directory**: commands run from the component directory.
   No `cd ..`, no paths outside the component except the shared
   workspace lock the package manager resolves itself.
6. **No network installs**: targets never install tools or download
   dependencies beyond what the locked environment already provides
   (`uv run`, `cargo`, the package manager's offline-capable run).
   Installing is the job of an optional `install` target.
7. **Exit codes**: a failing command fails the target. No `|| true`,
   no `-` prefix on recipe lines, no swallowed errors.
8. **Missing targets**: when a task's component lacks `test` or
   `check`, the task adds them (plan block, `<component>/Makefile` in
   `files_touched`). It never routes around a missing target.
9. **Components without executable code** (e.g. a plugin of markdown
   and YAML): keep the contract with minimal targets that validate
   what exists (schema/lint checks); `test` may run the same checks.
   Never an empty recipe that always passes.

## House style

Match existing component Makefiles in the repo before writing a new
one: same target names, same tool invocations.

```make
.PHONY: install test check format

test:
	uv run pytest

check:
	uv run ruff check <package> tests
```
