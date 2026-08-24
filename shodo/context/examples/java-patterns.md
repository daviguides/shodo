# Java Patterns and Best Practices

## Guard Clauses (Flat over Nested)

Validate and exit early — no arrow-shaped nesting:

```java
public Receipt processUser(UserId userId) {
    var user = repository.findUser(userId)
        .orElseThrow(() -> new UserNotFoundException(userId));

    if (!user.isActive()) {
        throw new UserInactiveException(userId);
    }

    var subscription = user.subscription()
        .orElseThrow(() -> new NoSubscriptionException(userId));

    // Main logic — flat and clear
    return process(user, subscription);
}
```

## Records with Validating Compact Constructors

Validation lives in the constructor — an existing instance is valid
by construction. Invalid states are unrepresentable:

```java
public record BranchName(String value) {
    public BranchName {
        if (value.isBlank() || value.contains(" ")) {
            throw new IllegalArgumentException(
                "invalid branch name: " + value);
        }
    }
}

public record Cents(long value) {
    public Cents {
        if (value < 0) {
            throw new IllegalArgumentException(
                "amount cannot be negative: " + value);
        }
    }
}
```

Domain newtypes make swapped arguments a compile error:

```java
Receipt transfer(AccountId from, AccountId to, Cents amount) { ... }
```

## Sealed Interface + Exhaustive Switch

```java
public sealed interface TriageOutcome
    permits Accepted, Deferred, Discarded {

    record Accepted(TaskId task) implements TriageOutcome {}
    record Deferred(LocalDate until) implements TriageOutcome {}
    record Discarded(String reason) implements TriageOutcome {}
}

// No default arm: adding a variant forces every switch to update
String describe(TriageOutcome outcome) {
    return switch (outcome) {
        case Accepted(TaskId task) -> "materialized as " + task;
        case Deferred(LocalDate until) -> "parked until " + until;
        case Discarded(String reason) -> "dropped: " + reason;
    };
}
```

## Typed Domain Exceptions

Small hierarchy per domain, always chained:

```java
public sealed class ConfigException extends RuntimeException {
    public ConfigException(String message, Throwable cause) {
        super(message, cause);
    }

    public static final class NotFound extends ConfigException { ... }
    public static final class Malformed extends ConfigException { ... }
}

public Config loadConfig(Path path) {
    if (!Files.exists(path)) {
        throw new ConfigException.NotFound(path);
    }
    try {
        return mapper.readValue(Files.readString(path), Config.class);
    } catch (IOException e) {
        throw new ConfigException.Malformed(
            "cannot parse config at " + path, e);
    }
}
```

## Optional at Return Boundaries

`Optional` for "absent is normal" returns — never fields, never
parameters:

```java
public Optional<User> findByEmail(Email email) { ... }

// Combinators over isPresent()/get()
var greeting = findByEmail(email)
    .map(User::name)
    .map("Hello, %s"::formatted)
    .orElse("Hello, guest");
```

## Streams over Index Loops

```java
// Declarative — no bounds errors, intent visible
List<String> activeNames = users.stream()
    .filter(User::isActive)
    .map(User::name)
    .toList();

// Extract a named predicate when the chain grows
private static boolean isProcessable(Item item) {
    return item.isValid()
        && item.status() == Status.ACTIVE
        && item.amount() > 100;
}

List<Output> results = items.stream()
    .filter(JavaPatterns::isProcessable)
    .map(this::process)
    .toList();
```

## Virtual Threads for I/O Fan-Out

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    List<Future<RepoStatus>> futures = repos.stream()
        .map(repo -> executor.submit(() -> git.status(repo)))
        .toList();

    for (Future<RepoStatus> future : futures) {
        report.add(future.get());
    }
}
```

Blocking style, thousands of concurrent I/O tasks, no reactive
machinery.

## Immutable Collections

```java
// Construction
List<String> statuses = List.of("pending", "active", "completed");
Map<String, Integer> limits = Map.of("wip", 3, "queue", 10);

// Pipelines end immutable
List<User> active = users.stream().filter(User::isActive).toList();

// Defensive copy in record constructors holding collections
public record Batch(List<Item> items) {
    public Batch {
        items = List.copyOf(items);
    }
}
```

## Orchestration Pattern

Public method orchestrates; private helpers in call order:

```java
public DiscountResult calculateDiscount(User user, Cents amount) {
    validateInputs(user, amount);

    var loyalty = loyaltyDiscount(user);
    var volume = volumeDiscount(amount);

    return combine(amount, loyalty, volume);
}

private void validateInputs(User user, Cents amount) { ... }

private Discount loyaltyDiscount(User user) { ... }

private Discount volumeDiscount(Cents amount) { ... }

private DiscountResult combine(Cents amount, Discount a, Discount b) { ... }
```

## Structured Logging (SLF4J 2 Fluent API)

```java
private static final Logger log = LoggerFactory.getLogger(SyncService.class);

log.atInfo()
    .addKeyValue("repo", repo.name())
    .addKeyValue("commits", pushed)
    .log("sync complete");
```

## Text Blocks for Queries

```java
String query = """
    SELECT u.id, u.name, u.email
    FROM users u
    WHERE u.is_active = :isActive
      AND u.created_at >= :startDate
    ORDER BY u.created_at DESC
    """;
```

## Javadoc Pattern

```java
/**
 * Scans {@code root} for git repositories with uncommitted changes.
 *
 * <p>Ignores hidden directories unless {@code options.includeHidden()}
 * is set.
 *
 * @throws ScanException if {@code root} does not exist or traversal
 *     fails
 */
public List<RepoEntry> scan(Path root, ScanOptions options) { ... }
```
