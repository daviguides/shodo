# Rust CLI Architecture Specification

Rust materialization of the shodo CLI architecture. Same layers and
dependency rules as the Python spec — the compiler enforces what
Python enforces by discipline.

## Package Structure

```
project-name/
├── Cargo.toml
├── src/
│   ├── main.rs              # Entrypoint — parse + dispatch only
│   ├── cli.rs               # clap derive: Cli struct + Command enum
│   ├── display.rs           # Semantic output (owo-colors, indicatif)
│   ├── error.rs             # Error types (thiserror)
│   │
│   ├── commands/            # Interface layer
│   ├── services/            # Business logic layer
│   ├── models/              # serde structs (data representation)
│   ├── integrations/        # Low-level I/O (fs, subprocess, APIs)
│   └── data/                # Bundled config (include_str!)
└── tests/
```

## Layered Architecture

Five layers with strict dependency direction:

```
commands/ → services/ → models/
                      → integrations/
                      → data/
```

### Layer Definitions

**`main.rs`** — Pure entrypoint. Calls `Cli::parse()`, matches the
command enum, dispatches to `commands/`. Zero logic. Under 40 lines.

**`cli.rs`** — clap derive definitions only. The `Cli` struct and
`Command` enum with doc comments as help text. No behavior.

**`commands/`** — Interface layer. One module per command. Translates
parsed args into service calls, formats output via `display`. Never
executes domain logic directly.

**`services/`** — Business logic. Pure functions and types operating
on domain concepts. **No clap imports. No terminal output.** Receives
typed data, returns `Result<T, Error>`.

**`models/`** — serde structs defining data shape. Pure representation
— derive `Serialize`/`Deserialize`, no logic, no I/O.

**`integrations/`** — Low-level operations. Filesystem, subprocess
wrappers (git, gh), YAML I/O, API clients. Domain-agnostic.

**`data/`** — Bundled config embedded at compile time via
`include_str!`. Not user-editable at runtime.

**`display.rs`** — Semantic output functions (`print_success`,
`print_error`, `print_warning`) over owo-colors/indicatif. Imported
by `commands/` only.

**`error.rs`** — thiserror enums. Services return typed errors;
`main.rs` maps them to exit codes and user-facing messages.

## Dependency Rules

| Layer | Can import from | Cannot import from |
|-------|----------------|-------------------|
| `commands/` | `services/`, `models/`, `display` | — |
| `services/` | `models/`, `integrations/`, `data/` | `commands/`, `display`, **clap** |
| `models/` | std, serde | anything else |
| `integrations/` | std, third-party | `commands/`, `services/`, `models/` |
| `display` | owo-colors, indicatif | `commands/`, `services/` |

## Entrypoint Convention

```rust
// main.rs
use clap::Parser;

use crate::cli::{Cli, Command};

fn main() -> anyhow::Result<()> {
    let cli = Cli::parse();

    match cli.command {
        Command::Scan(args) => commands::scan::run(args),
        Command::Sync(args) => commands::sync::run(args),
    }
}
```

```rust
// cli.rs
use clap::{Parser, Subcommand};

/// Git repo scanner: pull, status, and AI-powered commits.
#[derive(Parser)]
#[command(name = "rover", version)]
pub struct Cli {
    #[command(subcommand)]
    pub command: Command,
}

#[derive(Subcommand)]
pub enum Command {
    /// Scan repos for uncommitted changes.
    Scan(ScanArgs),
    /// Full sync: push + pull + rebase.
    Sync(SyncArgs),
}
```

## Sub-App Architecture (Context Separation)

The flat layout is the starting point. When command groups develop
distinct domains — services clustering by theme, models used by only
one group — graduate to **sub-apps**: vertical slices that replicate
the layers internally.

```
src/
├── main.rs              # Aggregator — dispatches to sub-apps
├── cli.rs               # Root Cli + top-level Command enum
├── display.rs           # Shared across all sub-apps
├── error.rs             # Shared error hierarchy
│
├── core/                # Sub-app for SHARED domain
│   ├── models/
│   ├── integrations/
│   └── services/
│
├── commit/              # Sub-app = vertical slice
│   ├── cli.rs           # Its own Subcommand enum
│   ├── commands/
│   ├── services/
│   └── models/
├── pull/                # Smaller slice — only the layers it needs
│   ├── cli.rs
│   ├── commands/
│   └── services/
└── sync/
```

### Sub-App Rules

1. **Each sub-app replicates only the layers it needs** — a slice
   with two layers is correct, not incomplete. Dependency rules apply
   WITHIN the slice.
2. **`core/` owns the shared domain** — sub-apps import from `core/`,
   **never from each other**. Logic needed by two slices moves to core.
3. **Top-level shared**: `main.rs`, `cli.rs`, `display.rs`, `error.rs`
   — transversal to all slices.
4. **Nested subcommands compose in `cli.rs`**:

```rust
#[derive(Subcommand)]
pub enum Command {
    #[command(subcommand)]
    Commit(commit::cli::CommitCommand),
    Pull(pull::cli::PullArgs),
    Sync(sync::cli::SyncArgs),
}
```

### When to Graduate

| Signal | Action |
|--------|--------|
| Command groups share no services | Split into sub-apps |
| `services/` clusters by theme | Each cluster becomes a slice |
| A model is used by one group only | It belongs inside that slice |
| Two slices need the same service | Move it to `core/` |

### Workspace Enforcement (Optional)

For strict boundaries, promote layers to workspace crates — the
dependency rules become compile errors, not review comments:

```toml
[workspace]
members = ["cli", "services", "models"]
```

`services/Cargo.toml` simply does not list clap — importing it is
impossible, not just forbidden.

## Cargo.toml Convention

```toml
[package]
name = "project-name"
version = "0.1.0"
edition = "2024"

[dependencies]
clap = { version = "4", features = ["derive"] }
serde = { version = "1", features = ["derive"] }
serde_yaml = "0.9"
anyhow = "1"
thiserror = "2"
owo-colors = "4"
indicatif = "0.17"

[dev-dependencies]
rstest = "0.24"

[lints.clippy]
all = { level = "deny", priority = -1 }

[[bin]]
name = "cli-name"
path = "src/main.rs"
```

## API Analogy

| API Concept (axum) | CLI Equivalent |
|--------------------|---------------|
| Router | `cli.rs` (Command enum) |
| handlers | `commands/` |
| services | `services/` |
| DTOs (serde) | `models/` |
| clients/repos | `integrations/` |
| static assets | `data/` (include_str!) |
| middleware/tracing | `display.rs` |
