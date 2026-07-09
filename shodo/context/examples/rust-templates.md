# Rust Code Templates

## Fallible Function Template

```rust
/// Brief description of what this function does.
///
/// # Errors
///
/// Returns `{ErrorType}::{Variant}` when {condition}.
pub fn {function_name}(
    {param}: &{Type},
    {options}: &{OptionsType},
) -> Result<{ReturnType}, {ErrorType}> {
    // Guard clauses first (flat > nested)
    if !{precondition} {
        return Err({ErrorType}::{Variant} { {context_field} });
    }

    let Some({value}) = {fallible_lookup} else {
        return Err({ErrorType}::{NotFoundVariant} { {context_field} });
    };

    // Main logic — propagate with ?
    let {intermediate} = {step_one}({value})?;
    let {result} = {step_two}(&{intermediate})?;

    Ok({result})
}
```

## Error Enum Template (thiserror)

```rust
#[derive(Debug, thiserror::Error)]
pub enum {Domain}Error {
    #[error("{human_readable_message} at {{path}}")]
    NotFound { path: PathBuf },

    #[error("{validation_message}: {{value}}")]
    Invalid { value: String },

    // Wrap lower-level errors — chain preserved automatically
    #[error("io failure during {operation}")]
    Io(#[from] std::io::Error),
}
```

## Module Structure Template

```rust
//! {Module purpose — one line}.
//!
//! {What this module owns and its boundaries.}

// Standard library
use std::path::Path;

// External crates
use serde::{Deserialize, Serialize};

// Local
use crate::models::{Model};

/// Main entry — orchestrates the module's workflow.
pub fn {main_function}(input: &{Input}) -> Result<{Output}, {Error}> {
    validate(input)?;
    let data = gather(input)?;
    Ok(build_output(data))
}

fn validate(input: &{Input}) -> Result<(), {Error}> { ... }

fn gather(input: &{Input}) -> Result<{Data}, {Error}> { ... }

fn build_output(data: {Data}) -> {Output} { ... }

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn {behavior_description}() { ... }
}
```

## CLI Entrypoint Template (clap)

```rust
// main.rs — parse + dispatch only, zero logic
use clap::Parser;

use crate::cli::{Cli, Command};

mod cli;
mod commands;
mod display;
mod error;
mod models;
mod services;

fn main() -> anyhow::Result<()> {
    let cli = Cli::parse();

    match cli.command {
        Command::Scan(args) => commands::scan::run(args),
        Command::Sync(args) => commands::sync::run(args),
    }
}
```

```rust
// cli.rs — clap derive definitions only
use clap::{Args, Parser, Subcommand};

/// {Tool description — becomes --help header}.
#[derive(Parser)]
#[command(name = "{cli-name}", version)]
pub struct Cli {
    #[command(subcommand)]
    pub command: Command,
}

#[derive(Subcommand)]
pub enum Command {
    /// {Command description — becomes subcommand help}.
    Scan(ScanArgs),
    /// {Command description}.
    Sync(SyncArgs),
}

#[derive(Args)]
pub struct ScanArgs {
    /// Include repos with no remote.
    #[arg(long)]
    pub all: bool,
}
```

```rust
// commands/scan.rs — interface layer: translate args, format output
use crate::cli::ScanArgs;
use crate::display;
use crate::services::scanner;

pub fn run(args: ScanArgs) -> anyhow::Result<()> {
    let report = scanner::scan_repos(args.all)?;
    display::print_report(&report);
    Ok(())
}
```

## Display Module Template

```rust
//! Semantic terminal output. Imported by commands/ only.

use owo_colors::OwoColorize;

pub fn print_success(message: &str) {
    println!("{} {message}", "✓".green());
}

pub fn print_error(message: &str) {
    eprintln!("{} {message}", "✗".red());
}

pub fn print_warning(message: &str) {
    eprintln!("{} {message}", "⚠".yellow());
}
```

## Test Module Template (rstest)

```rust
#[cfg(test)]
mod tests {
    use rstest::{fixture, rstest};

    use super::*;

    #[fixture]
    fn {fixture_name}() -> {Type} {
        {Type} { {field}: {value} }
    }

    #[rstest]
    fn {returns_expected_when_condition}({fixture_name}: {Type}) {
        let result = {function}(&{fixture_name});
        assert_eq!(result, {expected});
    }

    #[rstest]
    #[case({input_1}, {expected_1})]
    #[case({input_2}, {expected_2})]
    fn {behavior_across_cases}(
        #[case] input: {InputType},
        #[case] expected: {OutputType},
    ) {
        assert_eq!({function}(input), expected);
    }

    #[test]
    fn {returns_error_when_invalid}() {
        let result = {function}({invalid_input});
        assert!(matches!(result, Err({Error}::{Variant} { .. })));
    }
}
```

## Concrete Example: Repo Scanner Service

```rust
//! Repository discovery and dirty-state scanning.

use std::path::{Path, PathBuf};

use crate::core::integrations::git;
use crate::core::models::RepoEntry;

const DEFAULT_MAX_DEPTH: usize = 3;

#[derive(Debug, thiserror::Error)]
pub enum ScanError {
    #[error("scan root not found: {path}")]
    RootNotFound { path: PathBuf },

    #[error("io failure during scan")]
    Io(#[from] std::io::Error),
}

/// Scans `root` for git repositories with uncommitted changes.
///
/// # Errors
///
/// Returns `ScanError::RootNotFound` if `root` does not exist.
pub fn scan_dirty_repos(root: &Path) -> Result<Vec<RepoEntry>, ScanError> {
    if !root.exists() {
        return Err(ScanError::RootNotFound { path: root.to_path_buf() });
    }

    let repos = discover_repos(root, DEFAULT_MAX_DEPTH)?;

    Ok(repos
        .into_iter()
        .filter(|repo| git::has_uncommitted_changes(&repo.path))
        .collect())
}

fn discover_repos(root: &Path, max_depth: usize) -> Result<Vec<RepoEntry>, ScanError> {
    ...
}
```
