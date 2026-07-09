# Rust Style Specification

## Formatting Tools

### rustfmt Always
- **Format with `cargo fmt`** — default settings, no fights with the tool
- 100-column width (rustfmt default; the Rust convention — do not
  impose the Python 80 here)
- rustfmt owns line breaking, trailing commas, and indentation —
  never hand-format

### clippy as Law
- **`cargo clippy --all-targets -- -D warnings`** must pass — always
- Treat clippy suggestions as the idiomatic way, not as noise
- Silence a lint only with `#[allow(...)]` at the smallest scope,
  WITH a comment stating why

**Configuration pattern (`Cargo.toml`):**
```toml
[lints.rust]
unsafe_code = "forbid"

[lints.clippy]
all = { level = "deny", priority = -1 }
pedantic = "warn"
```

## Naming Conventions (RFC 430)

| Item | Convention |
|------|------------|
| Crates, modules, functions, variables | `snake_case` |
| Types, traits, enum variants | `PascalCase` |
| Constants, statics | `SCREAMING_SNAKE_CASE` |
| Lifetimes | short lowercase (`'a`, `'src`) |

### Conversion Method Semantics
The prefix declares the cost — follow it strictly:

| Prefix | Cost | Ownership |
|--------|------|-----------|
| `as_` | free | borrowed → borrowed |
| `to_` | expensive | borrowed → owned (new allocation) |
| `into_` | variable | owned → owned (consumes self) |

### Getters
- No `get_` prefix: `fn name(&self) -> &str`, not `fn get_name()`
- Booleans in question form: `is_active()`, `has_remote()`

## Module Organization

### Modern Layout (No mod.rs)
- **`foo.rs` + `foo/` directory** — never `foo/mod.rs`
- One module per domain concept; split when a module loses cohesion

**Pattern:**
```
src/
├── lib.rs           # Crate root — declares modules, minimal re-exports
├── scanner.rs       # Module file
└── scanner/         # Its submodules
    ├── discovery.rs
    └── report.rs
```

### Visibility
- Default private; open up deliberately (`pub(crate)` before `pub`)
- Re-export in `lib.rs` only the intended public API surface

## Multi-line Constructs

### Let rustfmt Decide
- Write the code; `cargo fmt` handles signature breaking and
  trailing commas
- Your job is **structure**, not layout: if a signature grows past
  ~4 parameters, that is a design smell — group into a struct

```rust
// ❌ Parameter creep
fn render(text: &str, color: Color, bold: bool, width: u16, wrap: bool)

// ✅ Options struct with Default
struct RenderOptions {
    color: Color,
    bold: bool,
    width: u16,
    wrap: bool,
}

fn render(text: &str, options: &RenderOptions)
```

## Code Spacing

### Blank Lines
- 1 blank line between items (functions, impls, type definitions)
- 1 blank line between logical sections inside a function
- No blank line padding at the start/end of blocks

## Comments

### Strategic Comments
- **English only**, explain **why**, not what
- Prefer self-documenting names and types over comments
- `// TODO(context):` / `// FIXME(context):` for tracked debt
- `///` doc comments for API docs; `//` only for internal reasoning

## Attribute Hygiene

- Derives on one line when short: `#[derive(Debug, Clone, PartialEq)]`
- `#[must_use]` on functions whose ignored return value is a bug
- `#[non_exhaustive]` on public enums expected to grow

## Consistency Enforcement

### Automated Tools
- `cargo fmt --check` and `cargo clippy -- -D warnings` in CI
- No manual formatting — always use tools
- `cargo doc --no-deps` must build without warnings (doc links resolve)
