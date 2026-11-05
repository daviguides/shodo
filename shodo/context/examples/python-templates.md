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
