# Rust Patterns and Best Practices

## Guard Clauses with let-else (Flat over Nested)

Validate and exit early — the Rust materialization of guard clauses:

```rust
fn process_user(user_id: UserId, db: &Database) -> Result<Receipt, ProcessError> {
    let Some(user) = db.find_user(user_id) else {
        return Err(ProcessError::UserNotFound { user_id });
    };

    if !user.is_active {
        return Err(ProcessError::UserInactive { user_id });
    }

    let Some(subscription) = user.subscription.as_ref() else {
        return Err(ProcessError::NoSubscription { user_id });
    };

    // Main logic — flat and clear
    process(&user, subscription)
}
```

## Error Propagation with `?`

```rust
fn load_config(path: &Path) -> Result<Config, ConfigError> {
    let raw = fs::read_to_string(path)?;   // io::Error → ConfigError via #[from]
    let config: Config = serde_yaml::from_str(&raw)?;
    config.validate()?;
    Ok(config)
}
```

## Typed Errors with thiserror

```rust
#[derive(Debug, thiserror::Error)]
pub enum ConfigError {
    #[error("config file not found at {path}")]
    NotFound { path: PathBuf },

    #[error("failed to read config")]
    Io(#[from] std::io::Error),

    #[error("invalid config: {0}")]
    Parse(#[from] serde_yaml::Error),

    #[error("max_retries must be <= {max}, got {value}")]
    RetriesOutOfRange { value: u32, max: u32 },
}
```

Callers match on variants — errors are API, not strings:

```rust
match load_config(&path) {
    Ok(config) => config,
    Err(ConfigError::NotFound { .. }) => Config::default(),
    Err(error) => return Err(error.into()),
}
```

## Newtype Pattern

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct RepoId(u64);

#[derive(Debug, Clone, PartialEq)]
pub struct BranchName(String);

impl BranchName {
    pub fn new(name: impl Into<String>) -> Result<Self, ValidationError> {
        let name = name.into();
        if name.is_empty() || name.contains(' ') {
            return Err(ValidationError::InvalidBranchName { name });
        }
        Ok(Self(name))
    }

    pub fn as_str(&self) -> &str {
        &self.0
    }
}
```

Validation lives in the constructor — an existing `BranchName` is
valid by construction. Invalid states are unrepresentable.

## Iterator Chains over Index Loops

```rust
// ✅ CORRECT — declarative, no bounds errors possible
let active_names: Vec<&str> = users
    .iter()
    .filter(|user| user.is_active)
    .map(|user| user.name.as_str())
    .collect();

// Extract a named predicate when the chain grows
fn is_processable(item: &Item) -> bool {
    item.is_valid() && item.status == Status::Active && item.amount > 100
}

let results: Vec<Output> = items
    .iter()
    .filter(|item| is_processable(item))
    .map(process)
    .collect();
```

## Exhaustive Match

```rust
#[derive(Debug)]
enum SyncPhase {
    Push,
    Pull,
    Rebase,
}

// ✅ No catch-all: adding SyncPhase::Prune forces this site to update
fn describe(phase: &SyncPhase) -> &'static str {
    match phase {
        SyncPhase::Push => "pushing dirty repos",
        SyncPhase::Pull => "pulling tracked repos",
        SyncPhase::Rebase => "rebasing worktrees",
    }
}
```

## Conversion Traits (From / TryFrom)

```rust
// Infallible: From
impl From<RepoEntry> for RepoSummary {
    fn from(entry: RepoEntry) -> Self {
        Self { name: entry.name, dirty: entry.has_changes() }
    }
}

// Fallible: TryFrom — never a panicking From
impl TryFrom<&str> for Email {
    type Error = ValidationError;

    fn try_from(value: &str) -> Result<Self, Self::Error> {
        if !value.contains('@') {
            return Err(ValidationError::InvalidEmail { value: value.into() });
        }
        Ok(Self(value.to_string()))
    }
}
```

## Options Struct with Default

```rust
#[derive(Debug, Clone)]
pub struct ScanOptions {
    pub include_hidden: bool,
    pub max_depth: usize,
    pub follow_symlinks: bool,
}

impl Default for ScanOptions {
    fn default() -> Self {
        Self { include_hidden: false, max_depth: 3, follow_symlinks: false }
    }
}

// Call sites stay readable and extensible:
scan(&root, &ScanOptions { max_depth: 5, ..Default::default() })
```

## Borrowed Parameters

```rust
// ✅ Accept the most general borrowed form
fn read_manifest(path: impl AsRef<Path>) -> Result<Manifest, ManifestError> {
    let raw = fs::read_to_string(path.as_ref())?;
    ...
}

fn total(amounts: &[u64]) -> u64 {          // NOT &Vec<u64>
    amounts.iter().sum()
}

fn shout(text: &str) -> String {            // NOT &String
    text.to_uppercase()
}
```

## Doc Comment Pattern

```rust
/// Scans `root` for git repositories with uncommitted changes.
///
/// Ignores hidden directories unless `options.include_hidden` is set.
///
/// # Errors
///
/// Returns `ScanError::RootNotFound` if `root` does not exist, or
/// `ScanError::Io` for filesystem failures during traversal.
pub fn scan(root: &Path, options: &ScanOptions) -> Result<Vec<RepoEntry>, ScanError> {
    ...
}
```
