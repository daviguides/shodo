# Python Code Templates

## Simple Function Template

```python
def {function_name}(
    {param1}: {Type1},
    {param2}: {Type2},
    {optional_param}: {Type3} = {default_value},
) -> {ReturnType}:
    """Brief description of what this function does.

    Args:
        param1: Description of param1
        param2: Description of param2
        optional_param: Description of optional parameter

    Returns:
        Description of return value

    Raises:
        ValueError: When input validation fails
        {SpecificError}: When specific condition occurs
    """
    # Validate inputs first (explicit > implicit)
    if not {validation_condition1}:
        raise ValueError("Clear error message about what's wrong")

    if not {validation_condition2}:
        raise ValueError("Another clear validation message")

    # Main logic (simple > complex)
    {main_logic}

    return {result}
```

## Class with Single Responsibility Template

```python
class {ClassName}:
    """Brief description of the class responsibility.

    This class handles {specific_responsibility} and provides
    {main_functionality} for {target_use_case}.
    """

    def __init__(
        self,
        {required_param}: {Type1},
        {optional_param}: {Type2} | None = None,
    ) -> None:
        """Initialize {ClassName} with required configuration."""
        self._validate_init_params(
            {required_param}={required_param},
            {optional_param}={optional_param},
        )

        self.{required_param} = {required_param}
        self.{optional_param} = {optional_param} or {default_value}

    def {main_public_method}(
        self,
        {param}: {Type},
    ) -> {ReturnType}:
        """Main public interface method."""
        self._validate_{main_public_method}_inputs({param}={param})

        {step1_result} = self._step1({param})
        {step2_result} = self._step2({step1_result})

        return self._finalize_result({step2_result})

    def _validate_init_params(
        self,
        {required_param}: {Type1},
        {optional_param}: {Type2} | None,
    ) -> None:
        """Validate constructor parameters."""
        if not {validation_condition}:
            raise ValueError("Clear validation message")

    def _validate_{main_public_method}_inputs(self, {param}: {Type}) -> None:
        """Validate inputs for main method."""
        if not {condition}:
            raise ValueError("Clear validation message")

    def _step1(self, {param}: {Type}) -> {IntermediateType}:
        """First step of the main process."""
        # Implementation
        return {result}

    def _step2(self, {input}: {IntermediateType}) -> {AnotherType}:
        """Second step of the main process."""
        # Implementation
        return {result}

    def _finalize_result(self, {input}: {AnotherType}) -> {ReturnType}:
        """Final step to prepare the result."""
        # Implementation
        return {final_result}
```

## Error Handling Template

```python
def {function_with_error_handling}(
    {param}: {Type},
) -> {ReturnType}:
    """Function that handles errors explicitly."""
    try:
        # Validate inputs first
        if not {validation_condition}:
            raise ValueError("Explicit validation message")

        # Attempt the operation
        result = {risky_operation}({param})

        # Validate result if needed
        if not {result_validation}:
            raise {SpecificError}("Result validation failed")

        return result

    except {SpecificExpectedException} as e:
        # Handle specific known exceptions
        logger.warning(f"Expected error in {function_name}: {e}")
        raise {WrappedError}("User-friendly message") from e

    except {AnotherSpecificException} as e:
        # Handle another specific case
        logger.error(f"Critical error in {function_name}: {e}")
        # Decide: re-raise, wrap, or handle
        raise

    except Exception as e:
        # Never let unexpected errors pass silently
        logger.error(f"Unexpected error in {function_name}: {e}")
        raise {WrappedError}(
            "An unexpected error occurred during {operation}"
        ) from e
```

## Module Structure Template

```python
# module_name.py
"""Module description.

This module provides functionality for {specific_purpose}
and handles {main_responsibility}.
"""

from decimal import Decimal
from typing import Protocol

from project.models import Model
from project.exceptions import CustomError


class RuleProtocol(Protocol):
    """Protocol for rule implementation."""

    def apply(self, data: DataType) -> ResultType:
        """Apply rule to data."""
        ...


def main_function(
    data: DataType,
    rules: list[RuleProtocol],
) -> ResultType:
    """Main function to process data."""
    # Implementation
    ...


def _helper_function(data: DataType) -> IntermediateType:
    """Helper to process specific step."""
    # Implementation
    ...


def _validate_inputs(data: DataType) -> None:
    """Helper to validate inputs."""
    # Implementation
    ...
```

## Concrete Examples

### User Discount Calculator Function

```python
def calculate_user_discount(
    user: User,
    purchase_amount: float,
    loyalty_years: int,
) -> float:
    """Calculate discount based on user loyalty and purchase amount."""
    if not user.is_active:
        raise ValueError("Cannot calculate discount for inactive user")

    if purchase_amount <= 0:
        raise ValueError("Purchase amount must be positive")

    if loyalty_years < 0:
        raise ValueError("Loyalty years cannot be negative")

    base_discount = 0.0
    if loyalty_years >= 5:
        base_discount = 0.15
    elif loyalty_years >= 2:
        base_discount = 0.10
    elif loyalty_years >= 1:
        base_discount = 0.05

    volume_bonus = min(purchase_amount / 1000 * 0.01, 0.10)
    total_discount = base_discount + volume_bonus

    return min(total_discount, 0.25)  # Cap at 25%
```

### User Discount Calculator Class

```python
class UserDiscountCalculator:
    """Calculates discounts for users based on various criteria."""

    def __init__(
        self,
        max_discount: float = 0.25,
        loyalty_tiers: dict[int, float] | None = None,
    ) -> None:
        self.max_discount = max_discount
        self.loyalty_tiers = loyalty_tiers or {
            1: 0.05,
            2: 0.10,
            5: 0.15,
        }

    def calculate_discount(
        self,
        user: User,
        purchase_amount: float,
    ) -> float:
        """Calculate final discount for user and purchase."""
        self._validate_inputs(user=user, purchase_amount=purchase_amount)

        loyalty_discount = self._calculate_loyalty_discount(user=user)
        volume_discount = self._calculate_volume_discount(
            purchase_amount=purchase_amount,
        )

        total_discount = loyalty_discount + volume_discount
        return min(total_discount, self.max_discount)

    def _validate_inputs(self, user: User, purchase_amount: float) -> None:
        """Validate inputs for discount calculation."""
        if not user.is_active:
            raise ValueError("Cannot calculate discount for inactive user")
        if purchase_amount <= 0:
            raise ValueError("Purchase amount must be positive")

    def _calculate_loyalty_discount(self, user: User) -> float:
        """Calculate discount based on user loyalty years."""
        loyalty_years = user.loyalty_years

        for min_years in sorted(self.loyalty_tiers.keys(), reverse=True):
            if loyalty_years >= min_years:
                return self.loyalty_tiers[min_years]

        return 0.0

    def _calculate_volume_discount(self, purchase_amount: float) -> float:
        """Calculate discount based on purchase volume."""
        volume_bonus = purchase_amount / 1000 * 0.01
        return min(volume_bonus, 0.10)
```

### Payment Processing with Error Handling

```python
def process_payment(
    payment_data: PaymentData,
    processor: PaymentProcessor,
) -> PaymentResult:
    """Process payment with explicit error handling."""
    try:
        # Validate inputs explicitly
        if not payment_data.amount > 0:
            raise ValueError("Payment amount must be positive")

        if not payment_data.currency:
            raise ValueError("Currency must be specified")

        # Process payment with clear error propagation
        result = processor.charge(
            amount=payment_data.amount,
            currency=payment_data.currency,
            source=payment_data.source,
        )

        if not result.success:
            raise PaymentProcessingError(
                f"Payment failed: {result.error_message}",
                error_code=result.error_code,
            )

        return result

    except PaymentNetworkError as e:
        # Explicitly handle network issues
        logger.error(f"Network error during payment: {e}")
        raise PaymentProcessingError(
            "Payment service temporarily unavailable",
        ) from e

    except PaymentValidationError as e:
        # Explicitly handle validation issues
        logger.warning(f"Payment validation failed: {e}")
        raise  # Re-raise validation errors as-is

    except Exception as e:
        # Never let unexpected errors pass silently
        logger.error(f"Unexpected error in payment processing: {e}")
        raise PaymentProcessingError(
            "An unexpected error occurred during payment processing",
        ) from e
```

## Domain-Specific Templates

### Machine Learning Model Training

```python
def train_model(
    dataset: pd.DataFrame,
    target_column: str,
    model_config: ModelConfig,
) -> TrainedModel:
    """Train ML model with explicit validation and error handling."""
    # Explicit validation
    if dataset.empty:
        raise ValueError("Dataset cannot be empty")

    # Clear preprocessing steps
    clean_data = preprocess_dataset(dataset)
    features, target = extract_features_and_target(
        data=clean_data,
        target_column=target_column,
    )

    # Explicit model training
    model = initialize_model(config=model_config)
    trained_model = model.fit(features=features, target=target)

    return trained_model
```

### Web API Endpoint

```python
@app.post("/users/{user_id}/discount")
def calculate_user_discount(
    user_id: int,
    discount_request: DiscountRequest,
    db: Database = Depends(get_database),
) -> DiscountResponse:
    """Calculate discount for specific user."""
    # Explicit validation
    user = get_user_or_404(user_id=user_id, db=db)
    validate_discount_request(request=discount_request)

    # Clear business logic
    discount = calculate_discount(
        user=user,
        amount=discount_request.amount,
    )

    # Explicit response
    return DiscountResponse(
        user_id=user_id,
        original_amount=discount_request.amount,
        discount_amount=discount.amount,
        final_amount=discount.final_amount,
    )
```
