# Python Style Specification

## Formatting Tools

### Ruff for Formatting
- **Format code and imports with Ruff** always
- **Never exceed 80 columns** width
- Configure Ruff in `pyproject.toml`

**Configuration pattern:**
```toml
[tool.ruff]
line-length = 80
target-version = "py313"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]
```

## Line Length

### 80 Column Maximum
- **80 columns maximum** for all code
- **Rationale**: Terminal compatibility and code review readability
- **Break long lines** in readable manner
- **Use trailing commas** to facilitate diffs

## PEP 8 Compliance

### Follow PEP 8 Conventions
- Use automated tools (Ruff) for enforcement
- Consistent indentation (4 spaces)
- Blank lines between definitions:
  - 2 blank lines before top-level functions/classes
  - 1 blank line between methods
- Whitespace around operators and after commas

## Naming Conventions

### Functions and Variables
- **snake_case** for functions and variables
- **UPPER_SNAKE_CASE** for constants
- **PascalCase** for classes

**Patterns:**
```python
# Functions
def calculate_user_discount(): ...
def validate_email_format(): ...

# Variables
user_discount_percentage = 0.15
max_retry_attempts = 3

# Constants
MAX_RETRY_ATTEMPTS = 3
API_BASE_URL = "https://api.example.com"

# Classes
class UserService: ...
class EmailValidator: ...
```

## Multi-line Constructs

### Function Signatures
- Break when more than one parameter
- Trailing comma after last parameter
- Each parameter on its own line

**Pattern:**
```python
def calculate_user_discount(
    user: User,
    purchase_amount: Decimal,
    loyalty_years: int,
    discount_type: str = "standard",
) -> Decimal:
    """Calculate discount based on user and purchase details."""
    ...
```

### Function Calls
- Break when more than one argument
- Use keyword arguments
- Trailing comma after last argument

**Pattern:**
```python
result = calculate_discount(
    user=current_user,
    amount=purchase_total,
    discount_type="premium",
    apply_taxes=True,
)
```

### Collections
- Use trailing commas in multi-line lists, dicts, sets
- Easier to add/remove items
- Better diffs in version control

**Pattern:**
```python
# Lists
items = [
    "first_item",
    "second_item",
    "third_item",
]

# Dictionaries
config = {
    "host": "localhost",
    "port": 8000,
    "debug": True,
}

# Sets
valid_statuses = {
    "pending",
    "active",
    "completed",
}
```

## Import Organization

### Three-Group Structure
1. Standard library
2. Third-party
3. Local

**Blank lines between groups**

**Pattern:**
```python
# Standard library
import json
import logging
from datetime import datetime
from pathlib import Path

# Third-party
import requests
from pydantic import BaseModel
from sqlalchemy import create_engine

# Local
from core.config import settings
from modules.user import UserService
from utils.helpers import format_currency
```

## Code Spacing

### Sparse Over Dense
- Use appropriate whitespace
- Don't compress logic into dense lines
- Prioritize readability

### Blank Lines
- 2 blank lines before top-level functions/classes
- 1 blank line between methods in a class
- 1 blank line between logical sections within functions

## Comments

### Strategic Comments
- **Comments in English** and well-formatted
- **Explain "why", not "what"**
- Prefer **self-documenting code** over excessive comments
- Use comments for:
  - Complex algorithms
  - Non-obvious business rules
  - Temporary workarounds (with TODO/FIXME)

## Docstrings

### Required Locations
- Module level (top of file)
- Class definitions
- Public functions and methods

### Format
- English only
- ≤ 80 columns width
- Break lines as needed
- Include Args, Returns, Raises sections when applicable

**Pattern:**
```python
def process_payment(
    user: User,
    amount: Decimal,
    payment_method: str,
) -> PaymentResult:
    """Process payment for user purchase.

    Args:
        user: User making the purchase
        amount: Payment amount in USD
        payment_method: Payment method identifier

    Returns:
        PaymentResult with transaction details

    Raises:
        ValidationError: If amount is invalid
        PaymentError: If payment processing fails
    """
    ...
```

## Consistency Enforcement

### Automated Tools
- Use Ruff for formatting and linting
- Configure pre-commit hooks
- Integrate with CI/CD pipeline
- No manual formatting - always use tools
