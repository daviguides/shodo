# Rust Language Specification

## Toolchain

### Required Version
- **Stable toolchain** always — never nightly features in production code
- **Edition 2024** for new crates
- Pin via `rust-toolchain.toml` when reproducibility matters

## Ownership and Borrowing

### Borrow by Default
- **Accept borrowed types** in parameters:
  - `&str` over `String`
  - `&[T]` over `Vec<T>`
  - `impl AsRef<Path>` over `PathBuf`
- **Take ownership only when storing** the value
- **Never `.clone()` to silence the borrow checker** — restructure instead
- Clone is legitimate when: crossing thread/task boundaries, the type is
  cheap (`Rc`/`Arc` bump), or a genuine copy is the semantic intent

**Pattern:**
```rust
// ✅ CORRECT - borrows, caller keeps ownership
fn format_greeting(name: &str) -> String {
    format!("Hello, {name}")
}

// ❌ WRONG - forces caller to move or clone
fn format_greeting(name: String) -> String { ... }
```

## Type System

### Newtypes Over Primitives
- Wrap domain values in **newtype structs** — never pass bare
  `String`/`u64` for domain concepts
- The compiler enforces what other languages need discipline for

```rust
struct UserId(u64);
struct Email(String);

fn send_welcome(user_id: UserId, email: &Email) -> Result<(), SendError> {
    ...
}
// Swapped arguments = compile error, not a production incident
```

### Exhaustive Matching
- **Match exhaustively** on enums you own — avoid `_` catch-all
- A new variant must FORCE every match site to be revisited

### Standard Derives
- `#[derive(Debug)]` on every type — always
- Add `Clone`, `PartialEq`, `Eq`, `Hash`, `Default` when semantics fit
- Implement `Display` for user-facing types (never rely on `Debug` output)

## Error Handling

### Result and the `?` Operator
- Return `Result<T, E>` from all fallible operations
- Propagate with `?` — **never `.unwrap()`/`.expect()` on recoverable paths**
- `.expect("reason")` is acceptable ONLY:
  - At binary startup for boot invariants
  - In tests
  - For provably impossible states (the message documents why)

### thiserror vs anyhow
- **Libraries**: typed error enums with `thiserror` — callers can match
- **Binaries**: `anyhow::Result` with `.context()` — humans read the chain

```rust
// Library crate
#[derive(Debug, thiserror::Error)]
pub enum ConfigError {
    #[error("config file not found at {path}")]
    NotFound { path: PathBuf },

    #[error("invalid config: {0}")]
    Parse(#[from] serde_yaml::Error),
}

// Binary crate
fn main() -> anyhow::Result<()> {
    let config = load_config(&path)
        .with_context(|| format!("failed to load {}", path.display()))?;
    ...
}
```

### Option Handling
- Use combinators (`map`, `and_then`, `unwrap_or_else`) or `let-else`
- Never `if x.is_some() { x.unwrap() }` — use `if let` / `let-else`

```rust
let Some(user) = find_user(id) else {
    return Err(UserError::NotFound { id });
};
```

## Dependency Management

### Cargo
- `cargo add` for dependencies — keep `Cargo.toml` sorted and minimal
- **Workspace** for multi-crate projects (shared `Cargo.lock`, unified lints)
- Enable only the crate features you use (`default-features = false`
  when it meaningfully cuts the dependency tree)

## Imports

### `use` Organization
- **Three groups** (blank line between): `std`, external crates, `crate`/local
- Merge paths from the same crate into one `use` tree

**Pattern:**
```rust
// Standard library
use std::collections::HashMap;
use std::path::PathBuf;

// External crates
use clap::Parser;
use serde::{Deserialize, Serialize};

// Local
use crate::models::Config;
use crate::services::scanner;
```

## Strings

### Formatting
- **Inline captured identifiers**: `format!("{name}")` over
  `format!("{}", name)`
- `Display` impls over ad-hoc `to_string` helpers

## Documentation

### Doc Comments Mandatory
- `//!` module docs at the top of every module
- `///` on every public item — compact, English, first line is a summary
- Document failure modes with `# Errors`; document `# Panics` if any

**Pattern:**
```rust
//! User authentication and session management.

/// Authenticates a user by email and password.
///
/// # Errors
///
/// Returns `AuthError::InvalidCredentials` if the password does not
/// match, `AuthError::UserNotFound` if the email is unknown.
pub fn authenticate(email: &Email, password: &str) -> Result<User, AuthError> {
    ...
}
```

## Constants

### No Magic Values
- `const` in `SCREAMING_SNAKE_CASE`, scoped to the module that owns them
- Environment-specific values via config structs, not hardcoded

```rust
const MAX_RETRY_ATTEMPTS: u32 = 3;
const DEFAULT_TIMEOUT: Duration = Duration::from_secs(30);
```

## Integration Philosophy

### Modern Rust Principles
- **Async-first** for I/O-bound work (tokio ecosystem)
- **Zero-cost abstractions** — iterators and traits over manual loops
  and dynamic dispatch, unless `dyn` is the clearer design
- **Make invalid states unrepresentable** — encode invariants in types,
  not in runtime checks
