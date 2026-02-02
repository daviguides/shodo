# Python Language Specification

## Python Version

### Required Version
- **Python ≥ 3.13** always
- **Never add `from __future__ import annotations`** or any other `__future__` imports
- **Always use native Python 3.13+ syntax**

## Type System

### Type Hints Mandatory
- **Use type hints always** in:
  - **Function signatures** (parameters and return values)
  - **Class variables**
  - **Other variables when needed for clarity**

### Modern Type Syntax (Python 3.13+)
```python
# ✅ CORRECT - Native Python 3.13+ syntax
list[str]              # NOT List[str]
dict[str, int]         # NOT Dict[str, int]
tuple[int, ...]        # NOT Tuple[int, ...]
set[float]             # NOT Set[float]
str | int              # NOT Union[str, int]
str | None             # NOT Optional[str]
```

### Function Signatures
- When function has **more than one argument**, **break into multiple lines**
- **Place comma after last argument** (trailing comma)

**Pattern:**
```python
def function_name(
    param1: Type1,
    param2: Type2,
    param3: Type3 = default_value,
) -> ReturnType:
    """Docstring here."""
    ...
```

## Dependency Management

### Package Manager
- **Use `uv` for all dependency management**
- **Never use `pip` or `poetry`**
- Maintain `pyproject.toml` updated

**Setup pattern:**
```bash
# Initialize project
uv init project_name
cd project_name

# Add dependencies
uv add requests pydantic

# Add dev dependencies
uv add --dev pytest black ruff mypy

# Run project
uv run python main.py
```

## Imports and Modules

### Import Organization
- **Order imports** in three groups (with blank lines between):
  1. **Standard library** (built-in)
  2. **Third-party** (uv/pip installed)
  3. **Local** (your project)

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

# Local
from core.config import settings
from modules.user import UserService
```

### Path Handling
- Prefer **`pathlib.Path`** over `os.path`
- Pathlib is more modern, readable, and cross-platform

**Pattern:**
```python
from pathlib import Path

config_file = Path("config") / "settings.yaml"
if config_file.exists():
    content = config_file.read_text(encoding="utf-8")
```

### Package Structure
- **Avoid empty `__init__.py`** files - not required since Python 3.3 (PEP 420)
- **Use `__init__.py` only for**:
  - Explicit exports
  - Module aliases
  - Package initialization logic

## String Formatting

### F-strings Always
- **Use f-strings exclusively**
- **Never use `%` or `.format()`**
- F-strings are more readable and performant

**Pattern:**
```python
# ✅ CORRECT
message = f"User {user.name} has {len(items)} items"
log_entry = f"Processing {batch_size} items at {datetime.now()}"
```

## Function Calls

### Keyword Arguments
- **Use keyword arguments (kwargs)** when **more than one argument**
- If only one argument, **use keyword when it increases clarity**
- **Break calls into multiple lines** when more than one argument
- **Always add trailing comma** after last argument

**Pattern:**
```python
# ✅ MULTIPLE ARGUMENTS - always kwargs and multiple lines
result = calculate_discount(
    user=current_user,
    amount=purchase_total,
    discount_type="premium",
    apply_taxes=True,
)

# ✅ SINGLE ARGUMENT - keyword when clearer
user = get_user(user_id=123)  # Clearer with keyword
data = serialize(obj)         # Obvious without keyword
```

## Documentation

### Docstrings Mandatory
- **Docstrings required** for:
  - **Modules** (top of file)
  - **Classes** (after `class`)
  - **Public functions** (after `def`)
- Must be **compact**, **in English**, **≤ 80 columns**
- **Break lines when necessary**

**Pattern:**
```python
"""User management module.

This module provides functionality for user authentication,
profile management, and user-related business logic.
"""

class UserService:
    """Service for user-related operations.

    Handles user creation, authentication, profile updates,
    and user data validation.
    """

    def authenticate_user(
        self,
        email: str,
        password: str,
    ) -> User | None:
        """Authenticate user with email and password.

        Args:
            email: User email address
            password: Plain text password

        Returns:
            User object if authentication successful, None otherwise

        Raises:
            ValidationError: If email format is invalid
        """
        ...
```

## Multi-line Strings

### SQL and Long Queries
- **Break long queries** (SQL, PromQL, etc.) into multiple lines
- Use triple quotes for complex queries

**Pattern:**
```python
query = """
    SELECT
        u.id,
        u.name,
        u.email
    FROM users u
    WHERE u.is_active = %(is_active)s
        AND u.created_at >= %(start_date)s
    ORDER BY u.created_at DESC
    LIMIT %(limit)s
"""
```

## Configuration and Constants

### Avoid Magic Values
- **Never use magic numbers/strings**
- Use **named constants** or **environment variables**
- Prefer configuration files for complex settings

**Pattern:**
```python
import os
from pathlib import Path

# Constants
MAX_RETRY_ATTEMPTS = 3
DEFAULT_TIMEOUT_SECONDS = 30
API_BASE_URL = os.getenv("API_BASE_URL", "https://api.example.com")

# Configuration
CONFIG_FILE = Path("config") / "app.yaml"
```

## Integration Philosophy

### Modern Python Principles
- **Async-first** when it makes sense
- **Performance-focused** libraries when available
- **Type hints support** mandatory
- **Modern syntax** always (Python 3.13+)
