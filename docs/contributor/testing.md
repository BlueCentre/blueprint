# Testing Guide

This guide covers testing practices and conventions for Blueprint.

## Overview

Blueprint uses Bazel for running tests across all languages. Tests are:
- **Fast** - Only re-run when code changes
- **Isolated** - Run in sandboxed environments
- **Cacheable** - Results cached for unchanged tests
- **Parallel** - Run simultaneously when possible

## Test Structure

### Test Organization

```
package/
├── BUILD
├── lib.py
├── lib_test.py          # Unit tests
└── integration_test.py  # Integration tests
```

### Naming Conventions

- Python: `test_*.py` or `*_test.py`
- Go: `*_test.go`
- JavaScript: `*.test.ts` or `*.spec.ts`
- Rust: `tests/` directory

## Writing Tests

### Python Tests

```python
import pytest
from mypackage import Calculator

def test_add():
    calc = Calculator()
    assert calc.add(2, 3) == 5

def test_add_negative():
    calc = Calculator()
    assert calc.add(-1, -2) == -3

@pytest.fixture
def calculator():
    return Calculator()

def test_with_fixture(calculator):
    assert calculator.add(1, 1) == 2
```

BUILD file:

```starlark
py_test(
    name = "test_calculator",
    srcs = ["test_calculator.py"],
    deps = [
        ":calculator",
        "@pip//pytest",
    ],
)
```

### Go Tests

```go
package calculator_test

import (
    "testing"
    "github.com/stretchr/testify/assert"
    "github.com/example/calculator"
)

func TestAdd(t *testing.T) {
    calc := calculator.New()
    result := calc.Add(2, 3)
    assert.Equal(t, 5, result)
}

func TestAddNegative(t *testing.T) {
    calc := calculator.New()
    result := calc.Add(-1, -2)
    assert.Equal(t, -3, result)
}
```

BUILD file:

```starlark
go_test(
    name = "calculator_test",
    srcs = ["calculator_test.go"],
    embed = [":calculator"],
    deps = ["@com_github_stretchr_testify//assert"],
)
```

### TypeScript Tests

```typescript
import { describe, it, expect } from 'vitest';
import { Calculator } from './calculator';

describe('Calculator', () => {
    it('should add two numbers', () => {
        const calc = new Calculator();
        expect(calc.add(2, 3)).toBe(5);
    });

    it('should handle negative numbers', () => {
        const calc = new Calculator();
        expect(calc.add(-1, -2)).toBe(-3);
    });
});
```

## Running Tests

### All Tests

```bash
bazel test //...
```

### Specific Tests

```bash
# Test a package
bazel test //path/to:all

# Test specific target
bazel test //path/to:test_name

# Test by tag
bazel test //... --test_tag_filters=unit
```

### Test Output

```bash
# Show all output
bazel test //path/to:test --test_output=all

# Show only errors
bazel test //path/to:test --test_output=errors

# Show summary
bazel test //... --test_summary=detailed
```

## Test Categories

### Unit Tests

Test individual functions/classes:

```python
def test_parse_date():
    result = parse_date("2024-01-15")
    assert result.year == 2024
    assert result.month == 1
    assert result.day == 15
```

Tag: `unit`

### Integration Tests

Test component interactions:

```python
def test_user_registration_flow():
    # Create user
    user = create_user("test@example.com")
    
    # Verify in database
    stored_user = db.get_user(user.id)
    assert stored_user.email == "test@example.com"
    
    # Verify email sent
    assert email_sent_to("test@example.com")
```

Tag: `integration`

### End-to-End Tests

Test complete workflows:

```bash
#!/bin/bash
# e2e_test.sh

# Start server
bazel run //server:app &
SERVER_PID=$!

# Wait for server
sleep 2

# Run tests
curl http://localhost:8080/health
# ... more tests

# Cleanup
kill $SERVER_PID
```

Tag: `e2e`

## Test Data

### Using Test Data

```starlark
py_test(
    name = "test_parser",
    srcs = ["test_parser.py"],
    data = [
        "testdata/valid.json",
        "testdata/invalid.json",
    ],
    deps = [":parser"],
)
```

Access in test:

```python
from rules_python.python.runfiles import runfiles

r = runfiles.Create()
data_path = r.Rlocation("_main/path/to/testdata/valid.json")
with open(data_path) as f:
    data = f.read()
```

### Generating Test Data

```starlark
genrule(
    name = "gen_test_data",
    outs = ["testdata.json"],
    cmd = "echo '{\"key\": \"value\"}' > $@",
)

py_test(
    name = "test",
    srcs = ["test.py"],
    data = [":gen_test_data"],
)
```

## Mocking

### Python - unittest.mock

```python
from unittest.mock import Mock, patch

def test_api_call():
    with patch('requests.get') as mock_get:
        mock_get.return_value.json.return_value = {'id': 1}
        
        result = fetch_user(1)
        
        assert result['id'] == 1
        mock_get.assert_called_once_with('/api/users/1')
```

### Go - testify/mock

```go
type MockDB struct {
    mock.Mock
}

func (m *MockDB) GetUser(id string) (*User, error) {
    args := m.Called(id)
    return args.Get(0).(*User), args.Error(1)
}

func TestFetchUser(t *testing.T) {
    mockDB := new(MockDB)
    mockDB.On("GetUser", "123").Return(&User{ID: "123"}, nil)
    
    service := NewService(mockDB)
    user, err := service.FetchUser("123")
    
    assert.NoError(t, err)
    assert.Equal(t, "123", user.ID)
    mockDB.AssertExpectations(t)
}
```

## Test Tags

Organize tests with tags:

```starlark
py_test(
    name = "fast_test",
    srcs = ["fast_test.py"],
    tags = ["unit", "fast"],
    deps = [":lib"],
)

py_test(
    name = "slow_test",
    srcs = ["slow_test.py"],
    tags = ["integration", "slow"],
    timeout = "long",
    deps = [":lib"],
)
```

Run by tag:

```bash
# Only unit tests
bazel test //... --test_tag_filters=unit

# Exclude slow tests
bazel test //... --test_tag_filters=-slow

# Multiple tags
bazel test //... --test_tag_filters=unit,integration
```

## Test Timeouts

Set appropriate timeouts:

```starlark
py_test(
    name = "quick_test",
    timeout = "short",  # 60 seconds
)

py_test(
    name = "normal_test",
    timeout = "moderate",  # 5 minutes (default)
)

py_test(
    name = "long_test",
    timeout = "long",  # 15 minutes
)

py_test(
    name = "very_long_test",
    timeout = "eternal",  # 60 minutes
)
```

## Flaky Tests

### Detecting Flaky Tests

```bash
# Run multiple times
bazel test //path/to:test --runs_per_test=10
```

### Handling Flaky Tests

```starlark
py_test(
    name = "flaky_test",
    srcs = ["flaky_test.py"],
    flaky = True,  # Allow up to 3 attempts
    deps = [":lib"],
)
```

Better: Fix the flakiness!

## Code Coverage

### Generate Coverage

```bash
# Python
bazel coverage //path/to:test

# View coverage report
genhtml bazel-out/_coverage/_coverage_report.dat -o coverage_html
open coverage_html/index.html
```

### Coverage Targets

```starlark
py_test(
    name = "test",
    srcs = ["test.py"],
    deps = [":lib"],
)

# Coverage is automatically tracked
```

## Performance Testing

### Benchmark Tests

```go
func BenchmarkAdd(b *testing.B) {
    calc := calculator.New()
    for i := 0; i < b.N; i++ {
        calc.Add(1, 2)
    }
}
```

Run:

```bash
bazel test //path/to:benchmark --test_arg=-test.bench=.
```

## Best Practices

1. **Test one thing** - Each test should verify one behavior
2. **Use descriptive names** - `test_user_creation_with_invalid_email`
3. **Follow AAA pattern** - Arrange, Act, Assert
4. **Keep tests fast** - Unit tests should be < 1 second
5. **Use fixtures** - Share setup code
6. **Mock external dependencies** - Database, API calls, etc.
7. **Test edge cases** - Empty inputs, null values, errors
8. **Write tests first** - TDD when appropriate
9. **Keep tests independent** - No shared state
10. **Clean up** - Remove temporary files, close connections

## Test Pyramid

Aim for this distribution:

```
         /\
        /E2E\       10% - End-to-end tests
       /------\
      /  Intg  \    20% - Integration tests
     /----------\
    /    Unit    \  70% - Unit tests
   /--------------\
```

## Continuous Testing

### Pre-commit

```bash
# Install hook
pre-commit install

# Tests run automatically on commit
git commit -m "Add feature"
```

### CI

All tests run on:
- Pull requests
- Pushes to main
- Nightly builds

## Resources

- [Testing Best Practices](https://bazel.build/reference/test-encyclopedia)
- [pytest Documentation](https://docs.pytest.org/)
- [Go Testing](https://go.dev/doc/tutorial/add-a-test)
- [Vitest Documentation](https://vitest.dev/)

## Next Steps

- Review [Code Style Guide](code-style.md)
- Check [Contributing Guide](contributing.md)
- Read [Development Setup](development.md)
