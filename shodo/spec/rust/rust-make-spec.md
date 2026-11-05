# Rust Make Targets

Contract: `spec/make-contract-spec.md`. Cargo uses the lockfile and the
local registry cache; pass `--locked` so a stale lock fails instead of
resolving new versions.

| Target | Commands |
|--------|----------|
| `test` | `cargo nextest run --locked --no-fail-fast --no-tests=warn`; `cargo test --doc --locked` when `src/lib.rs` exists |
| `check` | `cargo fmt --check`; `cargo clippy --locked --all-targets -- -D warnings` |

```make
.PHONY: install test check format

test:
	cargo nextest run --locked --no-fail-fast --no-tests=warn
	@if [ -f src/lib.rs ]; then cargo test --doc --locked; fi

check:
	cargo fmt --check
	cargo clippy --locked --all-targets -- -D warnings

format:
	cargo fmt
```

nextest is the runner (kinhin rust-tdd-spec); install with
`cargo install cargo-nextest --locked`; it does not run doctests, hence
the doc line (library crates only).

In a Cargo workspace member, run from the member directory; cargo
resolves the workspace lock itself. Add `--all-features` to every
command when the crate gates code behind features that CI must cover.
