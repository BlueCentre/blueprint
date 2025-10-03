# Python Guide

This guide covers working with Python in Blueprint using Aspect's `aspect_rules_py` and `rules_python`.

## Table of Contents

- [Setup](#setup)
- [Project Structure](#project-structure)
- [Dependencies](#dependencies)
- [Building and Running](#building-and-running)
- [Testing](#testing)
- [Linting and Formatting](#linting-and-formatting)
- [Advanced Topics](#advanced-topics)

## Setup

### Prerequisites

Python projects in Blueprint use:
- **Python 3.x** (version managed by Bazel)
- **pip/uv** for dependency resolution
- **pyproject.toml** for dependency declaration

### Initial Configuration

Blueprint is pre-configured for Python with:

```starlark
# MODULE.bazel
bazel_dep(name = "rules_python", version = "1.6.3")
bazel_dep(name = "aspect_rules_py", version = "1.6.3")
bazel_dep(name = "rules_uv", version = "0.88.0")
```

Configuration files:
- `pyproject.toml` - Project metadata and dependencies
- `requirements/` - Lock files for different environments
- `gazelle_python.yaml` - Gazelle configuration

## Project Structure

Typical Python package structure:

```
my-python-package/
├── BUILD
├── __init__.py
├── module.py
├── __main__.py
├── tests/
│   ├── BUILD
│   ├── __init__.py
│   └── test_module.py
└── README.md
```

### BUILD File Example

```starlark
load("@aspect_rules_py//py:defs.bzl", "py_library", "py_binary", "py_test")

py_library(
    name = "mylib",
    srcs = [
        "__init__.py",
        "module.py",
    ],
    visibility = ["//visibility:public"],
    deps = [
        "@pip//requests",
        "@pip//pydantic",
    ],
)

py_binary(
    name = "app",
    srcs = ["__main__.py"],
    main = "__main__.py",
    deps = [":mylib"],
)

py_test(
    name = "test_module",
    srcs = ["tests/test_module.py"],
    deps = [
        ":mylib",
        "@pip//pytest",
    ],
)
```

## Dependencies

### Adding Dependencies

1. Edit `pyproject.toml`:

```toml
[project]
name = "my_project"
version = "0.1.0"
dependencies = [
    "requests>=2.28.0",
    "pydantic>=2.0.0",
]
```

2. Update lock files:

```bash
./tools/repin
```

This runs `uv pip compile` to generate lock files in `requirements/`.

3. Update Gazelle manifest and BUILD files:

```bash
# Update manifest with installed packages
bazel run //:gazelle_python_manifest.update

# Generate BUILD files
bazel run //:gazelle
```

### Using Dependencies

Reference dependencies in BUILD files:

```starlark
py_library(
    name = "mylib",
    srcs = ["mylib.py"],
    deps = [
        "@pip//requests",           # External package
        "@pip//requests//:pkg",     # Alternative syntax
        "//other/package:lib",      # Internal dependency
    ],
)
```

### Development Dependencies

For test-only or dev dependencies:

```bash
# Add to requirements/test_requirements.in
pytest>=7.0.0
pytest-cov>=4.0.0

# Update lock files
./tools/repin
```

### Console Scripts

To use console scripts from packages:

```starlark
load("@rules_python//python/entry_points:py_console_script_binary.bzl", "py_console_script_binary")

py_console_script_binary(
    name = "black",
    pkg = "@pip//black",
)
```

## Building and Running

### Build a Library

```bash
bazel build //path/to:mylib
```

### Run a Binary

```bash
# Run with Bazel
bazel run //path/to:app

# With arguments
bazel run //path/to:app -- --arg1 value1

# With environment variables
bazel run //path/to:app --action_env=DEBUG=1
```

### Build Output Location

```bash
# Binary location
bazel build //path/to:app
ls -l bazel-bin/path/to/app

# Run directly
./bazel-bin/path/to/app
```

## Testing

### Writing Tests

Use pytest for testing:

```python
# tests/test_module.py
import pytest
from mypackage import module

def test_function():
    result = module.my_function()
    assert result == expected_value

def test_with_fixture(tmp_path):
    # Test with pytest fixtures
    pass

@pytest.mark.parametrize("input,expected", [
    (1, 2),
    (2, 4),
])
def test_parametrized(input, expected):
    assert module.double(input) == expected
```

### Running Tests

```bash
# Run all Python tests
bazel test //path/to:all

# Run specific test
bazel test //path/to:test_module

# With verbose output
bazel test //path/to:test_module --test_output=all

# With pytest arguments
bazel test //path/to:test_module --test_arg=-v --test_arg=-s
```

### Test Data

Include test data files:

```starlark
py_test(
    name = "test_with_data",
    srcs = ["test_with_data.py"],
    data = [
        "testdata/input.json",
        "testdata/expected.txt",
    ],
    deps = [":mylib"],
)
```

Access in test code:

```python
from rules_python.python.runfiles import runfiles

r = runfiles.Create()
data_path = r.Rlocation("_main/path/to/testdata/input.json")
with open(data_path) as f:
    data = f.read()
```

## Linting and Formatting

### Ruff

Blueprint uses Ruff for linting and formatting Python code.

Configuration in `pyproject.toml`:

```toml
[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "W", "I", "N"]
ignore = ["E501"]
```

### Formatting

```bash
# Format all Python files
format

# Format specific file
format path/to/file.py

# Check only (don't modify)
ruff check path/to/file.py
```

### Linting

```bash
# Lint all Python targets
aspect lint //...

# Lint specific target
aspect lint //path/to:target

# Autofix issues
aspect lint --fix //...
```

## Advanced Topics

### Virtual Environments

For local development with IDEs:

```bash
# Create venv (for IDE)
python -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -e .
```

**Note:** Bazel doesn't use this venv; it's only for IDE support.

### Type Checking with mypy

Add mypy configuration to `pyproject.toml`:

```toml
[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
```

Add to BUILD file:

```starlark
# Custom rule or aspect for mypy
# (Can integrate with rules_lint)
```

### Building Wheels

```starlark
load("@rules_python//python:packaging.bzl", "py_wheel")

py_wheel(
    name = "mypackage_wheel",
    distribution = "mypackage",
    version = "1.0.0",
    deps = [":mylib"],
)
```

Build:

```bash
bazel build //path/to:mypackage_wheel
```

### Docker/OCI Images

Blueprint supports building Python container images with pragmas:

```python
# __main__.py
# oci: build

def main():
    print("Hello from container!")

if __name__ == "__main__":
    main()
```

This automatically generates an `image` target:

```bash
# Build image
bazel build //path/to:image

# Load into Docker
bazel run //path/to:image

# Push to registry
bazel run //path/to:image.push
```

### Jupyter Notebooks

To work with notebooks (optional):

1. Add jupyter to `pyproject.toml`
2. Run `./tools/repin`
3. Create console script binary:

```starlark
py_console_script_binary(
    name = "jupyter",
    pkg = "@pip//jupyter",
)
```

4. Run:

```bash
bazel run //tools:jupyter -- notebook
```

### Multiple Python Versions

To test against multiple Python versions:

```starlark
py_test(
    name = "test_py311",
    srcs = ["test.py"],
    python_version = "3.11",
    deps = [":lib"],
)

py_test(
    name = "test_py312",
    srcs = ["test.py"],
    python_version = "3.12",
    deps = [":lib"],
)
```

### Cython Extensions

For Cython modules:

```starlark
# Custom rule or use rules_python cython support
# (Advanced - refer to rules_python docs)
```

## Common Patterns

### CLI Application

```python
# cli.py
import argparse

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--input", required=True)
    args = parser.parse_args()
    # ...

if __name__ == "__main__":
    main()
```

```starlark
py_binary(
    name = "cli",
    srcs = ["cli.py"],
    main = "cli.py",
    deps = [":lib"],
)
```

### FastAPI Web Application

```python
# app.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello World"}
```

```starlark
py_binary(
    name = "app",
    srcs = ["app.py"],
    main = "app.py",
    deps = [
        "@pip//fastapi",
        "@pip//uvicorn",
    ],
)
```

Run:

```bash
bazel run //path/to:app -- uvicorn app:app --reload
```

### Data Processing Script

```python
# process.py
import pandas as pd

def process_data(input_file, output_file):
    df = pd.read_csv(input_file)
    # Process...
    df.to_csv(output_file, index=False)

if __name__ == "__main__":
    process_data("input.csv", "output.csv")
```

```starlark
py_binary(
    name = "process",
    srcs = ["process.py"],
    data = ["input.csv"],
    deps = ["@pip//pandas"],
)
```

## Troubleshooting

### Module Not Found

```bash
# Update manifest
bazel run //:gazelle_python_manifest.update

# Regenerate BUILD files
bazel run //:gazelle
```

### Import Errors

Check that:
1. Package has `__init__.py`
2. BUILD file includes the source
3. Dependencies are listed in `deps`

### Lock File Out of Date

```bash
./tools/repin
```

### Gazelle Not Detecting Imports

Check `gazelle_python.yaml` configuration:

```yaml
manifest: gazelle_python.yaml
```

## Best Practices

1. **Use pyproject.toml** - Modern Python project configuration
2. **Pin dependencies** - Use lock files for reproducibility
3. **Type hints** - Add type annotations for better code quality
4. **Test coverage** - Aim for high test coverage
5. **Follow PEP 8** - Use Ruff for consistent style
6. **Virtual environments** - For IDE support only
7. **Avoid side effects** - Keep imports side-effect free

## Resources

- [aspect_rules_py Documentation](https://docs.aspect.build/rulesets/aspect_rules_py/)
- [rules_python Documentation](https://rules-python.readthedocs.io/)
- [Python Packaging Guide](https://packaging.python.org/)
- [Ruff Documentation](https://docs.astral.sh/ruff/)

## Next Steps

- Explore [Testing Guide](../contributor/testing.md)
- Learn about [Docker Images](../user/workflows.md#working-with-containers)
- Check [Troubleshooting](../user/troubleshooting.md#python-dependency-not-found)
