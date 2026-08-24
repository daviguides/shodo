# Java Anti-Patterns and Corrections

## Anti-Pattern 1: Null Returns Instead of Optional

### ❌ ANTI-PATTERN
```java
public User findUser(String email) {
    var user = repository.lookup(email);
    return user != null ? user : null;   // caller must remember to check
}
```

### ✅ CORRECT
```java
public Optional<User> findUser(Email email) {
    return repository.lookup(email);
}

// Caller handles absence explicitly
var name = findUser(email)
    .map(User::name)
    .orElseThrow(() -> new UserNotFoundException(email));
```

**Why this matters:**
- `null` hides absence in the signature; `Optional` declares it
- Under `@NullMarked`, NullAway makes the unchecked path a build error
- Absence handling becomes visible at every call site

## Anti-Pattern 2: Swallowed Exceptions

### ❌ ANTI-PATTERN
```java
try {
    return mapper.readValue(raw, Config.class);
} catch (Exception e) {
    return new Config();   // failure silently becomes a default
}
```

### ✅ CORRECT
```java
try {
    return mapper.readValue(raw, Config.class);
} catch (IOException e) {
    throw new ConfigException("invalid config at " + path, e);
}
```

**Why this matters:**
- The caller cannot distinguish "empty config" from "broken config"
- Chaining (`, e`) preserves the root cause for debugging
- Specific catch types keep unrelated failures loud

## Anti-Pattern 3: Lombok Where Records Belong

### ❌ ANTI-PATTERN
```java
@Data
@Builder
@AllArgsConstructor
public class UserDto {
    private String name;
    private String email;
}
```

### ✅ CORRECT
```java
public record UserDto(String name, Email email) {
    public UserDto {
        Objects.requireNonNull(name);
        Objects.requireNonNull(email);
    }
}
```

**Why this matters:**
- Records are language-level: no annotation processor, no
  internals-hacking that breaks on new JDKs
- Immutability and `equals`/`hashCode` are guaranteed, not generated
- Validation lives in the compact constructor — visible, debuggable

## Anti-Pattern 4: instanceof Cascades

### ❌ ANTI-PATTERN
```java
String describe(Object result) {
    if (result instanceof Synced) {
        return ((Synced) result).commits() + " commits";
    } else if (result instanceof Skipped) {
        return "skipped";
    }
    return "unknown";   // new variants land here silently
}
```

### ✅ CORRECT
```java
String describe(SyncResult result) {
    return switch (result) {
        case Synced(int commits) -> commits + " commits";
        case Skipped(String reason) -> "skipped: " + reason;
        case Failed(String branch, Throwable cause) -> "failed on " + branch;
    };
}
```

**Why this matters:**
- Sealed interface + no default arm = the compiler finds every
  switch when a variant is added
- Record deconstruction removes the cast noise
- "unknown" fallbacks turn design changes into silent bugs

## Anti-Pattern 5: Primitive Obsession

### ❌ ANTI-PATTERN
```java
void transfer(long from, long to, long amount) { ... }

transfer(amount, fromId, toId);   // compiles; corrupts data
```

### ✅ CORRECT
```java
record AccountId(long value) {}
record Cents(long value) {
    public Cents {
        if (value < 0) throw new IllegalArgumentException("negative: " + value);
    }
}

void transfer(AccountId from, AccountId to, Cents amount) { ... }

transfer(amount, fromId, toId);   // compile error
```

**Why this matters:**
- Three `long`s are interchangeable to the compiler; records are not
- Validation in the constructor makes invalid values unconstructible
- Signatures become self-documenting

## Anti-Pattern 6: Index Loops with Mutation

### ❌ ANTI-PATTERN
```java
List<String> dirty = new ArrayList<>();
for (int i = 0; i < repos.size(); i++) {
    if (repos.get(i).hasChanges()) {
        dirty.add(repos.get(i).name());
    }
}
```

### ✅ CORRECT
```java
List<String> dirty = repos.stream()
    .filter(RepoEntry::hasChanges)
    .map(RepoEntry::name)
    .toList();
```

**Why this matters:**
- No bounds arithmetic, no accumulator mutation
- The pipeline states intent: filter, then project
- `toList()` returns an immutable result

## Anti-Pattern 7: ThreadLocal + Thread Pools for I/O

### ❌ ANTI-PATTERN
```java
private static final ThreadLocal<RequestContext> CONTEXT = new ThreadLocal<>();
private final ExecutorService pool = Executors.newFixedThreadPool(200);
```

### ✅ CORRECT
```java
private static final ScopedValue<RequestContext> CONTEXT =
    ScopedValue.newInstance();

try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    ScopedValue.where(CONTEXT, requestContext)
        .run(() -> executor.submit(task));
}
```

**Why this matters:**
- Sized platform-thread pools cap I/O concurrency artificially;
  virtual threads scale to the workload
- `ThreadLocal` leaks across pooled threads; `ScopedValue` is
  immutable, bounded, and virtual-thread-cheap
- Blocking style stays simple — no reactive rewrite needed

## Anti-Pattern 8: Wildcard Imports and Stringly-Typed APIs

### ❌ ANTI-PATTERN
```java
import java.util.*;

public void setStatus(String status) {
    if (status.equals("actve")) { ... }   // typo compiles fine
}
```

### ✅ CORRECT
```java
import java.util.List;

public enum Status { PENDING, ACTIVE, COMPLETED }

public void setStatus(Status status) {
    if (status == Status.ACTIVE) { ... }
}
```

**Why this matters:**
- Enums make the value space finite and typo-proof
- Explicit imports document dependencies; wildcards hide them
- Switch over enums gets exhaustiveness checking for free

## Refactoring Example: Complete Transformation

### ❌ BEFORE (Multiple Anti-Patterns)
```java
public Double calc(Map<String, Object> u, Double amt, String t) {
    if (t == null) t = "standard";
    if (u.get("type").equals("premium")) {
        if (amt > 100) {
            if (t.equals("bulk")) return amt * 0.8;
            else return amt * 0.9;
        } else return amt * 0.95;
    } else {
        if (amt > 50) return amt * 0.95;
        else return amt;
    }
}
```

### ✅ AFTER (Shodō-Compliant)
```java
private static final BigDecimal PREMIUM_BULK_RATE = new BigDecimal("0.80");
private static final BigDecimal PREMIUM_RATE = new BigDecimal("0.90");
private static final BigDecimal SMALL_ORDER_RATE = new BigDecimal("0.95");

public BigDecimal calculateDiscountedPrice(
        User user, BigDecimal amount, DiscountType discountType) {
    if (amount.signum() <= 0) {
        throw new IllegalArgumentException("amount must be positive");
    }

    return switch (user.subscription()) {
        case PREMIUM -> premiumDiscount(amount, discountType);
        case STANDARD -> standardDiscount(amount);
    };
}

private BigDecimal premiumDiscount(BigDecimal amount, DiscountType type) {
    if (amount.compareTo(new BigDecimal("100")) <= 0) {
        return amount.multiply(SMALL_ORDER_RATE);
    }
    return switch (type) {
        case BULK -> amount.multiply(PREMIUM_BULK_RATE);
        case STANDARD -> amount.multiply(PREMIUM_RATE);
    };
}

private BigDecimal standardDiscount(BigDecimal amount) {
    return amount.compareTo(new BigDecimal("50")) > 0
        ? amount.multiply(SMALL_ORDER_RATE)
        : amount;
}
```

**Improvements:**
- ✅ Typed domain (`User`, enums) over `Map<String, Object>`
- ✅ `BigDecimal` for money — no floating-point drift
- ✅ Exhaustive switch over enum — no hidden else branch
- ✅ Guard clause validation, flat structure
- ✅ Named constants, no magic numbers
- ✅ Helpers in call order, single responsibility each
