# Python Anti-Patterns and Corrections

## Anti-Pattern 1: Magic Numbers/Strings

### ❌ ANTI-PATTERN
```python
def calculate_price(base_price):
    if base_price > 1000:
        return base_price * 0.9
    return base_price
```

### ✅ CORRECT
```python
BULK_DISCOUNT_THRESHOLD = 1000
BULK_DISCOUNT_RATE = Decimal("0.90")

def calculate_price(base_price: Decimal) -> Decimal:
    """Calculate final price with bulk discount if applicable."""
    if base_price > BULK_DISCOUNT_THRESHOLD:
        return base_price * BULK_DISCOUNT_RATE
    return base_price
```

**Why this matters:**
- Makes intentions explicit
- Easier to maintain and update
- Clear business rules

## Anti-Pattern 2: Deep Nesting (Arrow Anti-Pattern)

### ❌ ANTI-PATTERN
```python
def process_user(user):
    if user:
        if user.is_active:
            if user.has_subscription:
                if user.subscription.is_valid():
                    if user.subscription.plan == "premium":
                        return process_premium_user(user)
                    else:
                        return process_standard_user(user)
```

### ✅ CORRECT
```python
def process_user(user: User | None) -> ProcessResult:
    """Process user with clear validation flow."""
    # Guard clauses - validate and exit early
    if not user:
        raise ValueError("User cannot be None")

    if not user.is_active:
        raise UserNotActiveError("User account is not active")

    if not user.has_subscription:
        raise NoSubscriptionError("User has no subscription")

    if not user.subscription.is_valid():
        raise InvalidSubscriptionError("User subscription is not valid")

    # Main logic - flat and clear
    if user.subscription.plan == "premium":
        return process_premium_user(user)

    return process_standard_user(user)
```

**Why this matters:**
- Much easier to read and follow
- Each validation is clear
- Easy to add new validations
- Follows "Flat is better than nested"

## Anti-Pattern 3: Implicit Behavior

### ❌ ANTI-PATTERN
```python
def save_user(user, db=None):
    db = db or get_default_db()
    db.save(user)
```

### ✅ CORRECT
```python
def save_user(
    user: User,
    database: Database,
) -> None:
    """Save user to specified database.

    Args:
        user: User object to save
        database: Database connection to use for saving

    Raises:
        ValueError: If user is invalid
        DatabaseError: If save operation fails
    """
    if not user.is_valid():
        raise ValueError("Cannot save invalid user")

    database.save(user)

# Usage becomes explicit:
save_user(user=current_user, database=get_database())
```

**Why this matters:**
- No hidden defaults
- Explicit about dependencies
- Clear what's required vs optional

## Anti-Pattern 4: Silent Failures

### ❌ ANTI-PATTERN
```python
def parse_config(config_str):
    try:
        return json.loads(config_str)
    except:
        return {}
```

### ✅ CORRECT
```python
def parse_config(config_str: str) -> dict[str, Any]:
    """Parse configuration string into dictionary.

    Args:
        config_str: JSON string containing configuration

    Returns:
        Dictionary with parsed configuration

    Raises:
        ConfigParsingError: If config string is invalid JSON
        ValueError: If config_str is empty or None
    """
    if not config_str:
        raise ValueError("Configuration string cannot be empty")

    try:
        return json.loads(config_str)
    except json.JSONDecodeError as e:
        raise ConfigParsingError(
            f"Invalid JSON in configuration: {e}"
        ) from e
```

**Why this matters:**
- Errors are visible and debuggable
- Caller knows when something went wrong
- Includes context in error messages

## Anti-Pattern 5: Cryptic Names

### ❌ ANTI-PATTERN
```python
def calc(u, a):
    return a * 0.9 if u["premium"] else a
```

### ✅ CORRECT
```python
def calculate_discounted_price(user: User, amount: Decimal) -> Decimal:
    """Calculate final price with user-specific discount."""
    if user.subscription_type == "premium":
        return amount * Decimal("0.90")
    return amount
```

**Why this matters:**
- Self-documenting code
- No need to guess what `u` or `a` mean
- Clear intent

## Anti-Pattern 6: Dense One-Liners

### ❌ ANTI-PATTERN
```python
result = [process(x) for x in data if x.is_valid() and x.status == "active" and x.amount > 100]
```

### ✅ CORRECT
```python
# Filter first for clarity
active_valid_items = [
    item for item in data
    if item.is_valid()
    and item.status == "active"
    and item.amount > 100
]

# Process separately
result = [process(item) for item in active_valid_items]

# Or with helper function
def is_processable(item: Item) -> bool:
    """Check if item meets processing criteria."""
    return (
        item.is_valid()
        and item.status == "active"
        and item.amount > 100
    )

result = [process(item) for item in data if is_processable(item)]
```

**Why this matters:**
- Easier to understand logic
- Can add comments if needed
- Testable conditions

## Refactoring Example: Complete Transformation

### ❌ BEFORE (Multiple Anti-Patterns)
```python
def calc(u, amt, t=None):
    if t is None: t = "standard"
    if u["type"] == "premium":
        if amt > 100:
            if t == "bulk": return amt * 0.8
            else: return amt * 0.9
        else: return amt * 0.95
    else:
        if amt > 50: return amt * 0.95
        else: return amt
```

### ✅ AFTER (Zen-Compliant)
```python
def calculate_discounted_price(
    user: User,
    original_amount: Decimal,
    discount_type: str = "standard",
) -> Decimal:
    """Calculate final price after applying user-specific discounts."""
    if original_amount <= 0:
        raise ValueError("Original amount must be positive")

    if user.subscription_type == "premium":
        return _calculate_premium_discount(
            amount=original_amount,
            discount_type=discount_type,
        )

    return _calculate_standard_discount(amount=original_amount)


def _calculate_premium_discount(
    amount: Decimal,
    discount_type: str,
) -> Decimal:
    """Calculate discount for premium users."""
    if amount <= 100:
        return amount * Decimal("0.95")

    if discount_type == "bulk":
        return amount * Decimal("0.80")

    return amount * Decimal("0.90")


def _calculate_standard_discount(amount: Decimal) -> Decimal:
    """Calculate discount for standard users."""
    if amount > 50:
        return amount * Decimal("0.95")

    return amount
```

**Improvements:**
- ✅ Descriptive names
- ✅ Type hints
- ✅ Flat structure (guard clauses)
- ✅ Separated responsibilities
- ✅ No magic numbers (using Decimal for precision)
- ✅ Explicit validation
- ✅ Clear docstrings
