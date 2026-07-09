# Rust Anti-Patterns and Corrections

## Anti-Pattern 1: unwrap() on Recoverable Paths

### ❌ ANTI-PATTERN
```rust
fn load_config(path: &Path) -> Config {
    let raw = fs::read_to_string(path).unwrap();
    serde_yaml::from_str(&raw).unwrap()
}
```

### ✅ CORRECT
```rust
fn load_config(path: &Path) -> Result<Config, ConfigError> {
    let raw = fs::read_to_string(path)?;
    let config = serde_yaml::from_str(&raw)?;
    Ok(config)
}
```

**Why this matters:**
- A missing file is a recoverable condition, not a program bug
- `unwrap()` turns the caller's decision into your panic
- `?` costs one character more than `.unwrap()` and returns the error

## Anti-Pattern 2: clone() to Appease the Borrow Checker

### ❌ ANTI-PATTERN
```rust
fn report(entries: Vec<RepoEntry>) -> String {
    let names: Vec<String> = entries
        .iter()
        .map(|entry| entry.name.clone())
        .collect();
    let dirty = entries.iter().filter(|entry| entry.dirty).count();
    format!("{} repos, {dirty} dirty", names.len())
}
```

### ✅ CORRECT
```rust
fn report(entries: &[RepoEntry]) -> String {
    let names: Vec<&str> = entries
        .iter()
        .map(|entry| entry.name.as_str())
        .collect();
    let dirty = entries.iter().filter(|entry| entry.dirty).count();
    format!("{} repos, {dirty} dirty", names.len())
}
```

**Why this matters:**
- Cloning hides the design problem instead of solving it
- Borrows document intent: this function only READS entries
- When you genuinely need a clone, it should read as deliberate

## Anti-Pattern 3: Stringly-Typed Errors in Libraries

### ❌ ANTI-PATTERN
```rust
pub fn sync_repo(path: &Path) -> Result<(), String> {
    if !path.exists() {
        return Err(format!("path {} not found", path.display()));
    }
    ...
}
// Or equally bad in a library: Result<(), Box<dyn Error>>
```

### ✅ CORRECT
```rust
#[derive(Debug, thiserror::Error)]
pub enum SyncError {
    #[error("repo not found at {path}")]
    RepoNotFound { path: PathBuf },

    #[error("push rejected: {reason}")]
    PushRejected { reason: String },
}

pub fn sync_repo(path: &Path) -> Result<(), SyncError> {
    if !path.exists() {
        return Err(SyncError::RepoNotFound { path: path.to_path_buf() });
    }
    ...
}
```

**Why this matters:**
- String errors cannot be matched — callers can only print them
- Typed variants make error handling part of the API contract
- `anyhow` is for binaries (humans read); libraries expose types (code reads)

## Anti-Pattern 4: &String / &Vec<T> Parameters

### ❌ ANTI-PATTERN
```rust
fn shout(text: &String) -> String { ... }
fn total(amounts: &Vec<u64>) -> u64 { ... }
```

### ✅ CORRECT
```rust
fn shout(text: &str) -> String { ... }
fn total(amounts: &[u64]) -> u64 { ... }
```

**Why this matters:**
- `&str`/`&[T]` accept strictly more inputs (literals, slices, owned)
- `&String` adds a pointless double indirection
- clippy flags this (`ptr_arg`) — the idiom is unambiguous

## Anti-Pattern 5: Nested Match Pyramid

### ❌ ANTI-PATTERN
```rust
fn process(id: UserId, db: &Database) -> Result<Receipt, ProcessError> {
    match db.find_user(id) {
        Some(user) => match user.subscription.as_ref() {
            Some(subscription) => match subscription.validate() {
                Ok(()) => process_subscription(&user, subscription),
                Err(error) => Err(ProcessError::Invalid(error)),
            },
            None => Err(ProcessError::NoSubscription { id }),
        },
        None => Err(ProcessError::UserNotFound { id }),
    }
}
```

### ✅ CORRECT
```rust
fn process(id: UserId, db: &Database) -> Result<Receipt, ProcessError> {
    let Some(user) = db.find_user(id) else {
        return Err(ProcessError::UserNotFound { id });
    };

    let Some(subscription) = user.subscription.as_ref() else {
        return Err(ProcessError::NoSubscription { id });
    };

    subscription.validate().map_err(ProcessError::Invalid)?;

    process_subscription(&user, subscription)
}
```

**Why this matters:**
- `let-else` is the guard clause — validation reads top-to-bottom
- Each failure mode is visible at one indentation level
- The happy path ends the function instead of hiding in the center

## Anti-Pattern 6: Silently Discarding Results

### ❌ ANTI-PATTERN
```rust
let _ = fs::remove_file(&lock_path);
notify_webhook(&payload);  // Result ignored — compiler warns, dev ignores
```

### ✅ CORRECT
```rust
// Intentional discard — say WHY
if let Err(error) = fs::remove_file(&lock_path) {
    tracing::warn!(%error, path = %lock_path.display(), "stale lock cleanup failed");
}

// Or propagate when the caller must know
notify_webhook(&payload)?;
```

**Why this matters:**
- `let _ =` erases the only evidence a fallible call failed
- If the failure is truly ignorable, the log line documents the decision
- `#[must_use]` exists precisely to make this mistake loud

## Anti-Pattern 7: Primitive Obsession

### ❌ ANTI-PATTERN
```rust
fn transfer(from: u64, to: u64, amount: u64) -> Result<(), TransferError> { ... }

transfer(amount, from_id, to_id)  // compiles; corrupts data
```

### ✅ CORRECT
```rust
struct AccountId(u64);
struct Cents(u64);

fn transfer(from: AccountId, to: AccountId, amount: Cents) -> Result<(), TransferError> { ... }

transfer(amount, from_id, to_id)  // compile error
```

**Why this matters:**
- Three `u64`s are interchangeable to the compiler; newtypes are not
- The check costs nothing at runtime (zero-sized wrappers)
- Validation in the constructor makes invalid values unconstructible

## Anti-Pattern 8: Index Loops over Iterators

### ❌ ANTI-PATTERN
```rust
let mut dirty = Vec::new();
for i in 0..repos.len() {
    if repos[i].has_changes {
        dirty.push(repos[i].name.clone());
    }
}
```

### ✅ CORRECT
```rust
let dirty: Vec<&str> = repos
    .iter()
    .filter(|repo| repo.has_changes)
    .map(|repo| repo.name.as_str())
    .collect();
```

**Why this matters:**
- No bounds arithmetic to get wrong, no accidental clones
- The chain states intent: filter, then project
- Iterators optimize as well or better than manual loops
