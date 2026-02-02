# Python Library Preferences Specification

## CLI & Terminal
- **typer** - CLI with type hints (preferred over click/argparse)
- **rich** - Beautiful terminal output (preferred over plain print)

## Web & APIs
- **fastapi** - Async APIs with type hints (preferred over flask/django)

## Data Validation & Modeling
- **pydantic** - Validation, serialization, data models

## Testing
- **pytest** - Testing framework (preferred over unittest)
- **Classes to group test suites** - Organization pattern

## Data Processing
- **polars** - DataFrames (preferred over pandas)
  - **Rationale**: Performance and lazy evaluation

## LLM & AI
- **litellm** - Unified LLM APIs (preferred over direct openai)

## Database
- **motor** - MongoDB async (preferred over pymongo sync)

## Selection Philosophy

### Async-First
- Prefer async libraries when performance matters
- Use async for I/O-bound operations
- Maintain compatibility with asyncio ecosystem

### Type Hints Support
- **Type hints support mandatory**
- Libraries must provide type stubs or inline types
- Enables mypy checking and better IDE support

### Performance-Focused
- Choose performance-optimized libraries when available
- Consider lazy evaluation capabilities
- Benchmark critical paths

## Library Selection Criteria

### Priority Order
1. **Type hints support** - Must have
2. **Async support** - When doing I/O
3. **Performance** - When processing data
4. **Active maintenance** - Regular updates
5. **Community adoption** - Well-tested and documented

### Avoid
- Libraries without type hints
- Unmaintained libraries
- Libraries with complex/unclear APIs
- Sync-only libraries for I/O operations (when async alternatives exist)

## Integration Requirements

### Type Safety
- All chosen libraries must support type checking
- Must integrate with mypy
- Should provide IntelliSense support

### Documentation
- Prefer libraries with comprehensive documentation
- Clear examples and usage patterns
- Active community support

### Compatibility
- Must support Python ≥ 3.13
- Should use modern Python features
- No legacy Python 2 baggage
