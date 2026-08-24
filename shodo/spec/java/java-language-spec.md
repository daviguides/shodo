# Java Language Specification

## Java Version

### Required Version
- **Java 25 LTS** for all new code (GA Sep 2025)
- **Never target non-LTS releases** in production — feature releases
  (26, 27...) are for experimentation only
- Pin the JDK per project: Maven/Gradle toolchains + `.mise.toml`
  (or `.sdkmanrc`)

## Modern Language Features (Mandatory)

### Records for Data
- **Records for all immutable data carriers** — DTOs, value types,
  config, results
- Compact constructors for validation — an existing record instance
  is valid by construction
- **Never Lombok** — records + sealed types + pattern matching cover
  what Lombok patched; Lombok's internals-hacking breaks on new JDKs

```java
public record Email(String value) {
    public Email {
        if (value == null || !value.contains("@")) {
            throw new IllegalArgumentException(
                "invalid email: " + value);
        }
    }
}
```

### Sealed Interfaces + Pattern Matching
- **Sealed hierarchies** for closed domain variants — the compiler
  enforces exhaustiveness
- **Pattern matching for switch** with record deconstruction —
  no `instanceof` cascades, no default arm on sealed types

```java
public sealed interface SyncResult
    permits SyncResult.Synced, SyncResult.Skipped, SyncResult.Failed {

    record Synced(int commits) implements SyncResult {}
    record Skipped(String reason) implements SyncResult {}
    record Failed(String branch, Throwable cause) implements SyncResult {}
}

// Adding a variant FORCES every switch site to be revisited
String describe(SyncResult result) {
    return switch (result) {
        case SyncResult.Synced(int commits) -> commits + " commits pushed";
        case SyncResult.Skipped(String reason) -> "skipped: " + reason;
        case SyncResult.Failed(String branch, Throwable cause) ->
            "failed on " + branch;
    };
}
```

### Virtual Threads for I/O
- **Virtual threads for all I/O-bound concurrency** (final since 21,
  `synchronized` pinning fixed in 24)
- Blocking-style code on virtual threads over reactive frameworks
- `Executors.newVirtualThreadPerTaskExecutor()` in try-with-resources
- **Scoped Values** (final in 25) for immutable context — never
  `ThreadLocal` in new code
- **Structured Concurrency is still preview** — do NOT use in
  production code

### var Conventions (LVTI Style Guide)
- `var` when the type is obvious from the right-hand side
  (constructor, factory, literal)
- Explicit type when the initializer does not reveal it

```java
var users = new ArrayList<User>();      // ✅ obvious
var config = ConfigLoader.load(path);   // ❌ what type? — be explicit
Config config = ConfigLoader.load(path);
```

### Text Blocks
- Text blocks for SQL, JSON, and multi-line strings — never
  concatenated string fragments

## Null Safety

### JSpecify + Optional
- **`@NullMarked` at package level** (`package-info.java`) — JSpecify
  1.0 annotations, enforced by NullAway
- Under `@NullMarked`, unannotated types are non-null; nullable
  returns/fields are `@Nullable` — explicit and checked
- **`Optional<T>` for "absent is normal" return values** — never for
  fields or parameters
- Never return `null` from public methods — `Optional`, empty
  collection, or throw

```java
// package-info.java
@NullMarked
package com.example.tool.services;

import org.jspecify.annotations.NullMarked;
```

## Error Handling

### Exceptions Are the API
- **Unchecked domain exceptions** — a small sealed-style hierarchy
  per domain, extending `RuntimeException`
- Checked exceptions only at integration boundaries where the caller
  MUST decide (rare)
- **Always chain**: `throw new ConfigException("...", e)` — never
  swallow the cause
- Never `catch (Exception e) {}` — empty catch is a defect; log or
  rethrow, and say why when suppressing

```java
public class ConfigException extends RuntimeException {
    public ConfigException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### Fail Fast
- Validate at entry: `Objects.requireNonNull(x, "x")` for contract
  nulls, explicit `IllegalArgumentException` with context for values
- Record compact constructors carry domain validation

## Dependency Management

### Maven + Wrapper
- **Maven 3.9.x with the wrapper committed** (`./mvnw`) — the default
  for CLI tools and libraries
- Gradle (Kotlin DSL, wrapper committed) acceptable for large
  services — one tool per repo, never both
- BOM imports (`dependencyManagement`) for version alignment;
  minimal dependency tree — audit with `mvn dependency:tree`

## Imports

### Organization
- **No wildcard imports** — ever (`import java.util.*` hides intent)
- Static imports only for test assertions and well-known factories
  (`List.of` stays qualified)
- Ordering owned by the formatter (Spotless) — never hand-sorted

## Strings and Formatting

- `String.formatted()` or text blocks over `+` concatenation chains
- `StringBuilder` only in measured hot loops

## Documentation

### Javadoc Mandatory
- **Javadoc on every public type and public method** — compact,
  English, first sentence is the summary
- `@param`, `@return`, `@throws` when they add information
- Package `package-info.java` doubles as `@NullMarked` site and
  package doc

```java
/**
 * Authenticates a user by email and password.
 *
 * @throws AuthException if credentials are invalid or unknown
 */
public User authenticate(Email email, String password) { ... }
```

## Constants

### No Magic Values
- `static final` constants in `UPPER_SNAKE_CASE`, scoped to the class
  that owns them
- Environment-specific values via config records, never hardcoded

```java
private static final int MAX_RETRY_ATTEMPTS = 3;
private static final Duration DEFAULT_TIMEOUT = Duration.ofSeconds(30);
```

## Integration Philosophy

### Modern Java Principles
- **Data-oriented**: records + sealed interfaces + pattern matching
  model the domain; make invalid states unrepresentable
- **Virtual-threads-first** for I/O — simple blocking code, massive
  concurrency
- **Immutability by default** — `List.of`/`Map.of`, records,
  `Stream.toList()`; mutation is the documented exception
- **Streams over index loops** — declarative pipelines; extract named
  predicates when chains grow
