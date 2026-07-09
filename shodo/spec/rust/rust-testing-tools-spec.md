# Rust Testing Tools Specification

## Test Organization

### Unit Tests — Inline
- `#[cfg(test)] mod tests` at the bottom of the module under test
- Tests have access to private items — test the module's real internals

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn parses_valid_config() {
        let config = parse_config(VALID_YAML).unwrap();
        assert_eq!(config.max_retries, 3);
    }
}
```

### Integration Tests — `tests/`
- One file per public-API area in `tests/`
- Exercise the crate exactly as a consumer would (public API only)

### Naming
- Test names state the behavior: `returns_error_when_config_missing`,
  not `test_config_2`
- `.unwrap()`/`.expect()` are fine INSIDE tests — a panic is a failure

## Test Runner

### cargo-nextest Preferred
- **`cargo nextest run`** — parallel, isolated processes, better output
- `cargo test` remains the fallback (doctests still need it)

## Fixtures and Parametrization — rstest

```rust
use rstest::{fixture, rstest};

#[fixture]
fn sample_config() -> Config {
    Config { max_retries: 3, timeout: Duration::from_secs(30) }
}

#[rstest]
#[case("premium", 100, 90)]
#[case("premium", 1000, 800)]
#[case("standard", 100, 100)]
fn calculates_discount(
    #[case] tier: &str,
    #[case] amount: u64,
    #[case] expected: u64,
) {
    assert_eq!(calculate_discount(tier, amount), expected);
}
```

## Property-Based Testing — proptest

The hypothesis equivalent. Assert invariants over generated inputs:

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn roundtrip_preserves_config(retries in 0u32..100, secs in 1u64..3600) {
        let original = Config::new(retries, secs);
        let yaml = serde_yaml::to_string(&original).unwrap();
        let parsed: Config = serde_yaml::from_str(&yaml).unwrap();
        prop_assert_eq!(original, parsed);
    }
}
```

## Snapshot Testing — insta

For structured output (JSON, rendered text, error messages):

```rust
#[test]
fn renders_report() {
    let report = build_report(&sample_data());
    insta::assert_yaml_snapshot!(report);
}
```

Review changes with `cargo insta review`.

## Mocking — mockall

Mock **traits**, not concrete types — design services against traits
where substitution is needed:

```rust
#[cfg_attr(test, mockall::automock)]
trait GitClient {
    fn current_branch(&self) -> Result<String, GitError>;
}

#[test]
fn skips_sync_on_main() {
    let mut git = MockGitClient::new();
    git.expect_current_branch()
        .return_once(|| Ok("main".to_string()));

    let result = sync(&git);
    assert!(matches!(result, SyncResult::Skipped));
}
```

## Benchmarks — criterion

```rust
use criterion::{criterion_group, criterion_main, Criterion};

fn bench_scan(c: &mut Criterion) {
    c.bench_function("scan_1000_repos", |b| {
        b.iter(|| scan(std::hint::black_box(&repos)))
    });
}

criterion_group!(benches, bench_scan);
criterion_main!(benches);
```

## Coverage

### cargo-llvm-cov
```bash
cargo llvm-cov nextest --fail-under-lines 90
```

**Targets** (same bar as the Python spec):
- Overall: ≥ 90%
- Business logic (services): 100%

## Dev-Dependencies Pattern

```toml
[dev-dependencies]
rstest = "0.24"
proptest = "1"
insta = { version = "1", features = ["yaml"] }
mockall = "0.13"
criterion = "0.5"
```

## CI Commands

```bash
cargo fmt --check
cargo clippy --all-targets -- -D warnings
cargo nextest run
cargo test --doc
cargo llvm-cov nextest --fail-under-lines 90
```

## Best Practices

- One logical assertion per test; `assert_eq!` over bare `assert!`
- `matches!` for asserting enum variants:
  `assert!(matches!(err, ConfigError::NotFound { .. }))`
- Doctests on public API — examples that compile are documentation
  that cannot rot
- Keep test data minimal; build helpers/fixtures over copy-paste
