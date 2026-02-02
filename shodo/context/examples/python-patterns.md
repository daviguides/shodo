# Python Patterns and Best Practices

## Guard Clauses Pattern (Flat over Nested)

### Concept
Use early returns to validate preconditions and avoid deep nesting.

### Example
```python
def process_user(user: User | None) -> ProcessResult:
    """Process user with guard clause pattern."""
    # Guard clauses - validate and exit early
    if not user:
        raise ValueError("User cannot be None")

    if not user.is_active:
        raise UserError("User account is not active")

    if not user.has_subscription:
        raise SubscriptionError("User has no subscription")

    if not user.subscription.is_valid():
        raise InvalidSubscriptionError("Subscription is not valid")

    # Main logic here - flat and clear
    return process(user)
```

## Naming Patterns

### Functions (Verb-based)
```python
# ✅ EXPLICIT PATTERNS
def calculate_user_discount(user: User, amount: Decimal) -> Decimal: ...
def validate_email_format(email: str) -> bool: ...
def convert_celsius_to_fahrenheit(celsius: float) -> float: ...
def fetch_user_from_database(user_id: int, db: Database) -> User: ...
def transform_raw_data_to_model(data: dict) -> Model: ...
```

### Classes (Noun-based)
```python
# ✅ EXPLICIT PATTERNS
class EmailValidator: ...
class UserDiscountCalculator: ...
class PaymentProcessor: ...
class DataTransformer: ...
class ConfigurationManager: ...
```

### Booleans (Question-form)
```python
# ✅ CLEAR BOOLEAN INTENT
is_user_active: bool = True
has_valid_subscription: bool = False
can_process_payment: bool = True
should_send_notification: bool = False
```

### Constants (UPPER_SNAKE_CASE)
```python
# ✅ EXPLICIT CONSTANTS
MAX_RETRY_ATTEMPTS = 3
DEFAULT_TIMEOUT_SECONDS = 30
API_BASE_URL = "https://api.example.com"
BULK_DISCOUNT_THRESHOLD = 1000
```

## Validation Pattern

### Input Validation First
```python
def process_payment(amount: Decimal, currency: str) -> PaymentResult:
    """Process payment with upfront validation."""
    # Validate all inputs FIRST
    if amount <= 0:
        raise ValueError("Amount must be positive")

    if not currency:
        raise ValueError("Currency must be specified")

    if currency not in SUPPORTED_CURRENCIES:
        raise ValueError(f"Unsupported currency: {currency}")

    # Main logic after validation
    return _execute_payment(amount=amount, currency=currency)
```

## Orchestration Pattern

### Main Function as Orchestrator
```python
def calculate_discount(
    user: User,
    purchase_amount: Decimal,
) -> DiscountResult:
    """Main function orchestrates workflow."""
    # Main function coordinates steps
    _validate_inputs(user=user, purchase_amount=purchase_amount)

    loyalty_discount = _calculate_loyalty_discount(user=user)
    volume_discount = _calculate_volume_discount(amount=purchase_amount)

    total_discount = _combine_discounts(
        loyalty=loyalty_discount,
        volume=volume_discount,
    )

    return _create_result(
        original_amount=purchase_amount,
        discount=total_discount,
    )


def _validate_inputs(user: User, purchase_amount: Decimal) -> None:
    """Helper: Validate inputs."""
    ...


def _calculate_loyalty_discount(user: User) -> Decimal:
    """Helper: Calculate loyalty-based discount."""
    ...


def _calculate_volume_discount(amount: Decimal) -> Decimal:
    """Helper: Calculate volume-based discount."""
    ...
```

## Error Handling Patterns

### Specific Exceptions
```python
# Define specific exception types
class PaymentError(Exception):
    """Base exception for payment operations."""
    pass


class PaymentValidationError(PaymentError):
    """Raised when payment validation fails."""
    pass


class PaymentProcessingError(PaymentError):
    """Raised when payment processing fails."""
    pass


# Use in functions
def process_payment(data: PaymentData) -> PaymentResult:
    """Process payment with specific exceptions."""
    if not data.is_valid():
        raise PaymentValidationError(
            f"Invalid payment data: {data.validation_errors}"
        )

    try:
        result = charge_card(data)
    except NetworkError as e:
        raise PaymentProcessingError(
            "Payment service unavailable"
        ) from e

    return result
```

### Exception Chaining
```python
def parse_config(config_str: str) -> Config:
    """Parse configuration with exception chaining."""
    try:
        data = json.loads(config_str)
        return Config.from_dict(data)
    except json.JSONDecodeError as e:
        raise ConfigError(
            f"Invalid JSON in configuration: {e}"
        ) from e
    except ValidationError as e:
        raise ConfigError(
            f"Invalid configuration structure: {e}"
        ) from e
```

## Type Hints Patterns

### Function Signatures with Multiple Parameters
```python
def calculate_user_discount(
    user: User,
    purchase_amount: Decimal,
    loyalty_years: int,
    discount_type: str = "standard",
) -> Decimal:
    """Function with multiple parameters."""
    ...
```

### Union Types (Modern Syntax)
```python
# ✅ Python 3.13+ syntax
def process_result(value: str | int | None) -> str:
    """Handle multiple types."""
    ...

# ✅ Optional as union with None
def get_user(user_id: int) -> User | None:
    """May return None."""
    ...
```

### Collections
```python
# ✅ Python 3.13+ syntax
users: list[User] = []
config: dict[str, Any] = {}
valid_ids: set[int] = {1, 2, 3}
coordinates: tuple[float, float] = (1.0, 2.0)
```

## Keyword Arguments Pattern

### Multiple Arguments - Always Use kwargs
```python
# ✅ CORRECT - kwargs for multiple arguments
result = calculate_discount(
    user=current_user,
    amount=purchase_total,
    discount_type="premium",
    apply_taxes=True,
)

# ✅ CORRECT - single argument, keyword when clearer
user = get_user(user_id=123)

# ❌ WRONG - multiple args without keywords
result = calculate_discount(current_user, purchase_total, "premium", True)
```

## F-string Patterns

### String Formatting
```python
# ✅ ALWAYS use f-strings
name = "Alice"
age = 30
items = [1, 2, 3]

message = f"User {name} is {age} years old"
log_entry = f"Processing {len(items)} items"
error_msg = f"Invalid value: {value!r}"  # Use !r for repr

# For complex expressions, extract to variable first
discount_pct = discount * 100
message = f"Discount: {discount_pct:.2f}%"
```

## Pathlib Pattern

### File Operations
```python
from pathlib import Path

# ✅ CORRECT - Use pathlib
config_dir = Path("config")
config_file = config_dir / "settings.yaml"

if config_file.exists():
    content = config_file.read_text(encoding="utf-8")

# Create directories
output_dir = Path("output") / "reports"
output_dir.mkdir(parents=True, exist_ok=True)

# Iterate files
for file_path in config_dir.glob("*.yaml"):
    process_config(file_path)
```

## Multi-line Collection Pattern

### Lists, Dicts, Sets
```python
# ✅ CORRECT - trailing commas
users = [
    "alice@example.com",
    "bob@example.com",
    "carol@example.com",
]

config = {
    "host": "localhost",
    "port": 8000,
    "debug": True,
    "max_connections": 100,
}

valid_statuses = {
    "pending",
    "active",
    "completed",
    "archived",
}
```

## Import Organization Pattern

### Three-Group Structure
```python
# Group 1: Standard library
import json
import logging
from datetime import datetime
from pathlib import Path

# Group 2: Third-party
import requests
from pydantic import BaseModel
from sqlalchemy import create_engine

# Group 3: Local
from core.config import settings
from modules.user import UserService
from utils.helpers import format_currency
```

## Docstring Patterns

### Function Docstring
```python
def calculate_discount(
    user: User,
    amount: Decimal,
) -> Decimal:
    """Calculate discount for user purchase.

    Args:
        user: User object with subscription details
        amount: Purchase amount in USD

    Returns:
        Discount amount in USD

    Raises:
        ValueError: If amount is negative or zero
        UserError: If user account is invalid
    """
    ...
```

### Class Docstring
```python
class DiscountCalculator:
    """Calculate discounts based on user and purchase criteria.

    This class handles discount calculations for different user
    types and purchase volumes, applying tiered discount rules.

    Attributes:
        max_discount: Maximum allowed discount percentage
        loyalty_tiers: Mapping of loyalty years to discount rates
    """
    ...
```
