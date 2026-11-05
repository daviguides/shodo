# Rust Make Targets

Contract: `spec/make-contract-spec.md`. Cargo uses the lockfile and the
local registry cache; pass `--locked` so a stale lock fails instead of
resolving new versions.

| Target | Commands |
|--------|----------|
| `test` | `cargo nextest run --locked --no-fail-fast`; `cargo test --doc --locked` (library crates) |
| `check` | `cargo fmt --check`; `cargo clippy --locked --all-targets -- -D warnings` |

```make
.PHONY: install test check format

test:
	cargo nextest run --locked --no-fail-fast
	cargo test --doc --locked

check:
	cargo fmt --check
	cargo clippy --locked --all-targets -- -D warnings

format:
	cargo fmt
```

nextest is the runner (kinhin rust-tdd-spec): install it with
`cargo install cargo-nextest --locked`. It does not run doctests, so the
recipe adds `cargo test --doc`; drop that line in a binary-only crate
(no library target, `--doc` fails). A crate with no tests yet needs
`--no-tests=pass` on the nextest line.

In a Cargo workspace member, run from the member directory; cargo
resolves the workspace lock itself. Add `--all-features` to every
commands when the crate gates code behind features that CI must cover.
