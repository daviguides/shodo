# Rust Make Targets

Contract: `spec/make-contract-spec.md`. Cargo uses the lockfile and the
local registry cache; pass `--locked` so a stale lock fails instead of
resolving new versions.

| Target | Commands |
|--------|----------|
| `test` | `cargo test --locked` |
| `check` | `cargo fmt --check`; `cargo clippy --locked --all-targets -- -D warnings` |

```make
.PHONY: install test check format

test:
	cargo test --locked

check:
	cargo fmt --check
	cargo clippy --locked --all-targets -- -D warnings

format:
	cargo fmt
```

In a Cargo workspace member, run from the member directory; cargo
resolves the workspace lock itself. Add `--all-features` to both
commands when the crate gates code behind features that CI must cover.
