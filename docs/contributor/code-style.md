# Code Style Guide

This guide outlines the coding standards and conventions for Blueprint.

## General Principles

1. **Consistency** - Follow existing patterns
2. **Readability** - Write code for humans first
3. **Simplicity** - Prefer simple over clever
4. **Documentation** - Document complex logic
5. **Testing** - Write tests for all new code

## Language-Specific Guidelines

### Python

Follow PEP 8 with Ruff formatting:

```python
# Good
def calculate_total(items: list[Item]) -> float:
    """Calculate total price of items.
    
    Args:
        items: List of items to calculate total for
        
    Returns:
        Total price as float
    """
    return sum(item.price for item in items)


# Bad - missing types, docstring
def calc(items):
    total = 0
    for item in items:
        total = total + item.price
    return total
```

**Configuration:**

```toml
# pyproject.toml
[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "W", "I", "N"]
```

### Go

Follow official Go style:

```go
// Good
// ProcessOrders processes a batch of orders and returns any errors.
func ProcessOrders(ctx context.Context, orders []Order) error {
    for _, order := range orders {
        if err := processOrder(ctx, order); err != nil {
            return fmt.Errorf("processing order %s: %w", order.ID, err)
        }
    }
    return nil
}

// Bad - missing error wrapping, no context
func processOrders(orders []Order) error {
    for _, order := range orders {
        if err := processOrder(order); err != nil {
            return err
        }
    }
    return nil
}
```

**Use:**
- `gofmt` for formatting
- Error wrapping with `%w`
- Context for cancellation
- Interfaces for testability

### JavaScript/TypeScript

Follow TypeScript best practices:

```typescript
// Good
interface User {
    id: string;
    name: string;
    email: string;
}

async function fetchUser(id: string): Promise<User> {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
        throw new Error(`Failed to fetch user: ${response.statusText}`);
    }
    return response.json();
}

// Bad - any types, no error handling
async function getUser(id) {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
}
```

**Configuration:**

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

### Rust

Follow Rust conventions:

```rust
// Good
/// Calculates the fibonacci number at position n.
///
/// # Arguments
/// * `n` - The position in the fibonacci sequence
///
/// # Examples
/// ```
/// assert_eq!(fibonacci(5), 5);
/// ```
pub fn fibonacci(n: u32) -> u32 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

// Bad - no docs, inefficient
pub fn fib(n: u32) -> u32 {
    if n == 0 { return 0; }
    if n == 1 { return 1; }
    fib(n-1) + fib(n-2)
}
```

## Bazel Style

### BUILD Files

```starlark
# Good - organized, commented
load("@rules_python//python:defs.bzl", "py_library", "py_test")

# Library with clear dependencies
py_library(
    name = "mylib",
    srcs = [
        "lib.py",
        "utils.py",
    ],
    visibility = ["//visibility:public"],
    deps = [
        "@pip//requests",
        "//shared:common",
    ],
)

# Test with clear setup
py_test(
    name = "test_mylib",
    srcs = ["test_mylib.py"],
    data = ["testdata/input.json"],
    deps = [
        ":mylib",
        "@pip//pytest",
    ],
)
```

**Conventions:**
- Load statements at top
- Group by type (libraries, binaries, tests)
- Use descriptive target names
- Add visibility explicitly
- Sort dependencies alphabetically

### Starlark

```starlark
# Good - clear, documented
def custom_rule(name, srcs, deps = [], **kwargs):
    """Creates a custom build rule.
    
    Args:
        name: Target name
        srcs: Source files
        deps: Dependencies (default: [])
        **kwargs: Additional arguments
    """
    native.genrule(
        name = name,
        srcs = srcs,
        outs = [name + ".out"],
        cmd = "process $< > $@",
        **kwargs
    )
```

## Documentation

### Code Comments

```python
# Good - explains why, not what
# Use exponential backoff to handle rate limits
time.sleep(2 ** retry_count)

# Bad - explains what (obvious)
# Sleep for 2 to the power of retry_count
time.sleep(2 ** retry_count)
```

### Docstrings

**Python:**

```python
def process_data(input_file: str, output_file: str, validate: bool = True) -> int:
    """Process data from input file and write to output file.
    
    Reads CSV data, validates if requested, and writes results.
    
    Args:
        input_file: Path to input CSV file
        output_file: Path to output file
        validate: Whether to validate data (default: True)
        
    Returns:
        Number of records processed
        
    Raises:
        FileNotFoundError: If input file doesn't exist
        ValueError: If data validation fails
        
    Example:
        >>> count = process_data("in.csv", "out.csv")
        >>> print(f"Processed {count} records")
    """
```

**Go:**

```go
// ProcessData reads CSV data, validates it, and writes results.
//
// The function reads from inputFile, optionally validates each record,
// and writes processed results to outputFile.
//
// Parameters:
//   - ctx: Context for cancellation
//   - inputFile: Path to input CSV file
//   - outputFile: Path to output file
//   - validate: Whether to validate data
//
// Returns the number of records processed and any error encountered.
func ProcessData(ctx context.Context, inputFile, outputFile string, validate bool) (int, error)
```

### README Files

Each package should have a README:

```markdown
# Package Name

Brief description of what this package does.

## Usage

```python
from mypackage import MyClass

obj = MyClass()
result = obj.process()
```

## API

### `MyClass`

Main class for processing.

**Methods:**
- `process()` - Processes data and returns result

## Testing

```bash
bazel test //path/to:test
```
```

## Naming Conventions

### Python

```python
# Modules: lowercase_with_underscores
import my_module

# Classes: PascalCase
class UserManager:
    pass

# Functions/methods: lowercase_with_underscores
def process_order():
    pass

# Constants: UPPERCASE_WITH_UNDERSCORES
MAX_RETRIES = 3

# Private: _leading_underscore
def _internal_helper():
    pass
```

### Go

```go
// Exported: PascalCase
type UserManager struct {}

func ProcessOrder() {}

// Unexported: camelCase
type internalHelper struct {}

func processHelper() {}

// Constants: PascalCase
const MaxRetries = 3
```

### JavaScript/TypeScript

```typescript
// Classes/Interfaces: PascalCase
class UserManager {}
interface User {}

// Functions/variables: camelCase
function processOrder() {}
const userName = "John";

// Constants: UPPER_CASE
const MAX_RETRIES = 3;

// Private: #prefix (or _prefix for TypeScript)
class Example {
    #privateField: string;
}
```

## Error Handling

### Python

```python
# Good - specific exceptions
try:
    result = risky_operation()
except FileNotFoundError as e:
    logger.error(f"File not found: {e}")
    return None
except ValueError as e:
    logger.error(f"Invalid value: {e}")
    raise

# Bad - bare except
try:
    result = risky_operation()
except:
    pass
```

### Go

```go
// Good - check and wrap errors
func ProcessFile(path string) error {
    file, err := os.Open(path)
    if err != nil {
        return fmt.Errorf("opening file %s: %w", path, err)
    }
    defer file.Close()
    // ...
    return nil
}

// Bad - ignore errors
func ProcessFile(path string) {
    file, _ := os.Open(path)
    // ...
}
```

### TypeScript

```typescript
// Good - typed errors
async function fetchUser(id: string): Promise<User> {
    try {
        const response = await fetch(`/api/users/${id}`);
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        return await response.json();
    } catch (error) {
        if (error instanceof Error) {
            throw new Error(`Failed to fetch user: ${error.message}`);
        }
        throw error;
    }
}
```

## Testing

### Test Names

```python
# Good - descriptive
def test_user_creation_with_valid_email():
    pass

def test_user_creation_fails_with_invalid_email():
    pass

# Bad - vague
def test_user():
    pass
```

### Test Structure

Follow Arrange-Act-Assert:

```python
def test_calculate_total():
    # Arrange
    items = [Item(price=10), Item(price=20)]
    calculator = Calculator()
    
    # Act
    total = calculator.calculate_total(items)
    
    # Assert
    assert total == 30
```

## Git Commits

Follow conventional commits:

```bash
# Good
feat(python): add support for Python 3.12
fix(build): resolve issue with npm dependencies
docs(readme): update installation instructions

# Bad
Update code
Fix bug
WIP
```

Format:
```
type(scope): subject

body

footer
```

**Types:** feat, fix, docs, style, refactor, test, chore

## Code Review

### What to Look For

- [ ] Correct functionality
- [ ] Tests included
- [ ] Documentation updated
- [ ] Code follows style guide
- [ ] No security issues
- [ ] Appropriate error handling
- [ ] Reasonable performance

### Giving Feedback

```markdown
# Good - constructive
Consider using a switch statement here for better readability:
```go
switch status {
case "active":
    return true
case "inactive":
    return false
default:
    return false
}
```

# Bad - not helpful
This code is bad.
```

## Formatting

Use automated formatters:

```bash
# Format everything
format

# Python
ruff format .

# Go
gofmt -w .

# JavaScript/TypeScript
prettier --write .

# Rust
cargo fmt
```

## Linting

Use configured linters:

```bash
# Run all linters
aspect lint //...

# Python
ruff check .

# Go
golangci-lint run

# JavaScript/TypeScript
eslint .

# Rust
cargo clippy
```

## Resources

- [PEP 8](https://peps.python.org/pep-0008/) - Python style guide
- [Effective Go](https://go.dev/doc/effective_go) - Go style guide
- [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)
- [Rust Style Guide](https://doc.rust-lang.org/nightly/style-guide/)
- [Bazel Style Guide](https://bazel.build/rules/bzl-style)

## Next Steps

- Review [Contributing Guide](contributing.md)
- Check [Testing Guide](testing.md)
- Read [Architecture Overview](architecture.md)
