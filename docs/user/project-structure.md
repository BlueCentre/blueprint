# Project Structure

This document explains the organization and structure of the Blueprint repository.

## Root Directory Layout

```
blueprint/
├── .aspect/              # Aspect CLI configuration
│   └── cli/             # Custom CLI extensions
├── .bazelrc             # Bazel configuration flags
├── .bazelversion        # Bazel version specification
├── .devcontainer/       # VS Code dev container config
├── .github/             # GitHub Actions and workflows
├── .vscode/             # VS Code settings
├── BUILD                # Root BUILD file
├── MODULE.bazel         # Bazel module dependencies
├── REPO.bazel          # Repository configuration
├── docs/                # Documentation (you are here!)
├── requirements/        # Python package requirements
├── tools/               # Build and development tools
├── package.json         # Node.js package configuration
├── pyproject.toml       # Python project configuration
├── go.mod               # Go module dependencies
├── Cargo.toml           # Rust package configuration
└── README.md            # Main project README
```

## Key Files and Their Purpose

### Bazel Configuration Files

#### `MODULE.bazel`

The central configuration file for Bazel modules (bzlmod). Defines:

- External dependencies (rules_python, rules_go, rules_js, etc.)
- Language toolchains (Python, Go, JavaScript, Rust, Java, etc.)
- Module extensions for package managers (pip, pnpm, cargo, maven)

**Key sections:**
```starlark
# Language rules
bazel_dep(name = "rules_python", version = "1.6.3")
bazel_dep(name = "rules_go", version = "0.57.0")
bazel_dep(name = "aspect_rules_js", version = "2.6.0")
bazel_dep(name = "rules_rust", version = "0.63.0")

# Developer experience
bazel_dep(name = "aspect_rules_lint", version = "1.9.0")
bazel_dep(name = "bazel_env.bzl", version = "0.5.0")
```

#### `.bazelrc`

Bazel command-line options and configurations. Includes:

- Build optimization flags
- Remote cache settings
- Platform-specific configurations
- Release/stamping configurations

Imports curated presets from `bazelrc-preset.bzl`.

#### `.bazelversion`

Specifies the exact Bazel version to use. Bazelisk reads this file to download and run the correct version.

#### `BUILD` Files

Define build targets in each package. The root `BUILD` file includes:

- Gazelle configuration for code generation
- Package linking for npm/pip
- Shared configuration exports (eslint, prettier, etc.)

### Developer Environment

#### `.envrc`

Direnv configuration that:

- Adds Bazel-managed tools to PATH
- Automatically runs when entering the directory
- Requires `direnv allow` to activate

#### `tools/`

Development and build tools:

```
tools/
├── BUILD                    # Tool targets
├── bazel_env/              # Environment setup
├── format/                 # Formatting configurations
├── lint/                   # Linting configurations
├── oci/                    # Container image tools
├── platforms/              # Platform definitions
├── tools.lock.json         # Locked tool versions
├── tools.go                # Go tool dependencies
├── preset.bazelrc          # Bazel preset flags
├── repin                   # Dependency repinning script
└── workspace_status.sh     # Build stamping script
```

### Language-Specific Files

#### Python

- **`pyproject.toml`** - Python project metadata and dependencies
- **`requirements/`** - Requirements files for different environments
  - `requirements_lock.txt` - Locked runtime dependencies
  - `test_requirements_lock.txt` - Test dependencies
- **`gazelle_python.yaml`** - Gazelle Python configuration

#### JavaScript/TypeScript

- **`package.json`** - Node.js project metadata
- **`pnpm-lock.yaml`** - pnpm lockfile
- **`pnpm-workspace.yaml`** - pnpm workspace configuration
- **`eslint.config.mjs`** - ESLint configuration
- **`prettier.config.cjs`** - Prettier configuration

#### Go

- **`go.mod`** - Go module dependencies
- **`go.sum`** - Go module checksums

#### Rust

- **`Cargo.toml`** - Rust package configuration
- **`Cargo.lock`** - Rust dependency lockfile (if present)

#### Java/Kotlin

- **`maven_install.json`** - Maven dependency lock
- **`pmd.xml`** - PMD (Java linter) rules
- **`ktlint-baseline.xml`** - Kotlin lint baseline

### Aspect CLI Configuration

#### `.aspect/cli/config.yaml`

Main Aspect CLI configuration:

```yaml
configure:
  languages:
    go: true
    javascript: true
    python: true
  plugins:
    - .aspect/cli/shell.star
    - .aspect/cli/package-json-scripts.star
    - .aspect/cli/pytest_main.star
    - .aspect/cli/go_image.star
    - .aspect/cli/py3_image.star
```

#### `.aspect/cli/*.star`

Starlark plugins for code generation and automation:

- **`shell.star`** - Generates targets for shell scripts
- **`package-json-scripts.star`** - Creates Bazel targets from npm scripts
- **`pytest_main.star`** - Generates pytest entry points
- **`py3_image.star`** - Creates Python OCI images
- **`go_image.star`** - Creates Go OCI images

### Editor Configuration

#### `.editorconfig`

Cross-editor configuration for:

- Indentation style (spaces vs tabs)
- Line endings
- Charset
- Trailing whitespace

#### `.vscode/`

VS Code specific settings:

- Bazel extension configuration
- Python interpreter settings
- Recommended extensions

#### `.devcontainer/`

Development container configuration for:

- Reproducible development environments
- Pre-configured tools and extensions
- GitHub Codespaces support

## Package Structure

Blueprint encourages organizing code into packages:

```
my-package/
├── BUILD                    # Build definitions
├── src/                     # Source code
│   ├── lib.py              # Python library
│   ├── main.go             # Go binary
│   └── index.ts            # TypeScript entry point
├── test/                    # Tests
│   ├── test_lib.py         # Python tests
│   ├── main_test.go        # Go tests
│   └── index.test.ts       # TypeScript tests
└── README.md               # Package documentation
```

### BUILD File Structure

Typical BUILD file:

```starlark
load("@aspect_rules_py//py:defs.bzl", "py_library", "py_test")
load("@rules_go//go:def.bzl", "go_library", "go_test")

# Python targets
py_library(
    name = "mylib",
    srcs = ["lib.py"],
    visibility = ["//visibility:public"],
)

py_test(
    name = "test_lib",
    srcs = ["test_lib.py"],
    deps = [":mylib"],
)

# Go targets
go_library(
    name = "mygolib",
    srcs = ["main.go"],
    importpath = "github.com/example/project/my-package",
    visibility = ["//visibility:public"],
)
```

## Directory Conventions

### Private vs Public

- Directories starting with `_` are typically internal/private
- Use `visibility` attribute in BUILD files to control access

### Test Files

- Python: `*_test.py` or `test_*.py`
- Go: `*_test.go`
- JavaScript/TypeScript: `*.test.ts` or `*.spec.ts`
- Rust: `tests/` directory or inline with `#[test]`

### Generated Files

Bazel generates files in:

- `bazel-bin/` - Build outputs
- `bazel-out/` - Intermediate build artifacts
- `bazel-testlogs/` - Test logs

These are symlinks and should not be committed to version control.

## Configuration File Precedence

Understanding configuration priority:

1. Command-line flags (highest priority)
2. `.bazelrc` in workspace root
3. User's `~/.bazelrc`
4. System-wide `/etc/bazel.bazelrc` (lowest priority)

## Next Steps

- Learn about [Developer Workflows](workflows.md)
- Understand [Architecture](../contributor/architecture.md)
- Explore [Language-Specific Guides](../languages/README.md)
