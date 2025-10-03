# Architecture Overview

This document provides a high-level overview of Blueprint's architecture and design principles.

## Table of Contents

- [Design Principles](#design-principles)
- [System Architecture](#system-architecture)
- [Build System Architecture](#build-system-architecture)
- [Developer Experience Architecture](#developer-experience-architecture)
- [Language Integration](#language-integration)
- [Extension Points](#extension-points)

## Design Principles

Blueprint is built on these core principles:

### 1. **Polyglot First**

Support multiple programming languages with a unified build system and consistent developer experience.

### 2. **Zero Config**

Provide sensible defaults and automatic code generation to minimize manual configuration.

### 3. **Reproducible**

Ensure builds are deterministic and reproducible across different machines and environments.

### 4. **Fast Feedback**

Optimize for quick build times, incremental compilation, and rapid iteration.

### 5. **Extensible**

Allow customization and extension without modifying core infrastructure.

### 6. **Production Ready**

Include best practices for formatting, linting, testing, and deployment.

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Blueprint                            │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                  Developer Interface                 │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐    │   │
│  │  │   CLI    │  │  direnv  │  │  IDE Extensions  │    │   │
│  │  └──────────┘  └──────────┘  └──────────────────┘    │   │
│  └──────────────────────────────────────────────────────┘   │
│                            │                                │
│  ┌──────────────────────────────────────────────────────┐   │
│  │            Developer Experience Layer                │   │
│  │  ┌─────────────┐  ┌────────────┐  ┌───────────────┐  │   │
│  │  │ Aspect CLI  │  │ Gazelle    │  │  bazel_env    │  │   │
│  │  └─────────────┘  └────────────┘  └───────────────┘  │   │
│  │  ┌─────────────┐  ┌────────────┐  ┌───────────────┐  │   │
│  │  │ rules_lint  │  │ Formatters │  │ Pre-commit    │  │   │
│  │  └─────────────┘  └────────────┘  └───────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
│                            │                                │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Bazel Build System                      │   │
│  │  ┌──────────────────────────────────────────────┐    │   │
│  │  │         Module System (bzlmod)               │    │   │
│  │  │  - MODULE.bazel                              │    │   │
│  │  │  - External dependencies                     │    │   │
│  │  │  - Toolchain registration                    │    │   │
│  │  └──────────────────────────────────────────────┘    │   │
│  │  ┌──────────────────────────────────────────────┐    │   │
│  │  │         Build Rules & Actions                │    │   │
│  │  │  - Language rules (py, go, js, rust, etc.)   │    │   │
│  │  │  - Build actions & caching                   │    │   │
│  │  └──────────────────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────┘   │
│                            │                                │
│  ┌──────────────────────────────────────────────────────┐   │
│  │            Language Ecosystems                       │   │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐         │   │
│  │  │ Python │ │   Go   │ │   JS   │ │  Rust  │  ...    │   │
│  │  │  pip   │ │ go.mod │ │  pnpm  │ │ Cargo  │         │   │
│  │  └────────┘ └────────┘ └────────┘ └────────┘         │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Build System Architecture

### Bazel Core

Bazel is the foundation, providing:

- **Action Graph** - DAG of build actions
- **Dependency Analysis** - Determines what needs rebuilding
- **Sandboxed Execution** - Isolated, reproducible builds
- **Caching** - Local and remote artifact caching
- **Parallel Execution** - Multi-core build parallelism

### Module System (bzlmod)

Blueprint uses Bazel's modern module system:

```
MODULE.bazel
    │
    ├─── Language Rules
    │    ├─── rules_python
    │    ├─── rules_go
    │    ├─── aspect_rules_js
    │    ├─── rules_rust
    │    ├─── rules_java
    │    └─── rules_kotlin
    │
    ├─── Developer Tools
    │    ├─── aspect_rules_lint
    │    ├─── bazel_env.bzl
    │    ├─── bazelrc-preset.bzl
    │    └─── rules_multitool
    │
    └─── Package Managers
         ├─── pip (Python)
         ├─── pnpm (JavaScript)
         ├─── go_deps (Go)
         └─── crates (Rust)
```

### Build Phases

```
1. Loading Phase
   ├─── Parse BUILD files
   ├─── Evaluate Starlark
   └─── Build action graph

2. Analysis Phase
   ├─── Resolve dependencies
   ├─── Apply aspects (for linting)
   └─── Determine actions to run

3. Execution Phase
   ├─── Check cache for artifacts
   ├─── Run actions in sandbox
   └─── Cache outputs
```

## Developer Experience Architecture

### Tool Management (bazel_env.bzl)

```
tools/tools.lock.json
    │
    ↓
bazel run //tools:bazel_env
    │
    ↓
bazel-out/bazel_env-opt/bin/tools/bazel_env/bin/
    ├─── copier
    ├─── yq
    ├─── pnpm
    └─── gazelle (symlink)
    │
    ↓
direnv (.envrc)
    │
    ↓
Tools available on $PATH
```

**Flow:**
1. Tools defined in `tools.lock.json`
2. `bazel_env` downloads and extracts tools
3. Tools placed in known directory
4. `direnv` adds directory to PATH
5. Tools available in shell

### Code Generation (Gazelle)

```
Source Files
    │
    ↓
Gazelle Extensions
    ├─── Go: analyze imports
    ├─── Python: analyze imports + manifest
    └─── JavaScript: analyze package.json
    │
    ↓
Generate BUILD files
    ├─── Library targets
    ├─── Binary targets
    ├─── Test targets
    └─── Dependencies
```

### Aspect CLI Extensions

Custom Starlark plugins in `.aspect/cli/`:

```
.aspect/cli/config.yaml
    │
    ├─── shell.star
    │    └─── Generates sh_binary/sh_test
    │
    ├─── package-json-scripts.star
    │    └─── Generates js_binary from scripts
    │
    ├─── pytest_main.star
    │    └─── Generates pytest entry points
    │
    ├─── py3_image.star
    │    └─── Generates OCI images for Python
    │
    └─── go_image.star
         └─── Generates OCI images for Go
```

**Plugin Flow:**
1. Plugin analyzes source files
2. Matches patterns or pragmas
3. Generates BUILD targets
4. Targets available to Bazel

### Linting Architecture

Blueprint uses `rules_lint` with Bazel aspects:

```
Source Files
    │
    ↓
Bazel Aspect (rules_lint)
    │
    ├─── Python → ruff
    ├─── JavaScript → eslint
    ├─── Go → nogo (staticcheck)
    ├─── Shell → shellcheck
    ├─── C/C++ → clang-tidy
    ├─── Java → pmd
    └─── Kotlin → ktlint
    │
    ↓
Report Files
    │
    ↓
aspect lint (collects & presents)
```

**Benefits:**
- Incremental linting (only changed files)
- Cached lint results
- Parallel execution
- Consistent configuration

## Language Integration

### Python Integration

```
pyproject.toml
    │
    ↓
./tools/repin (uv pip compile)
    │
    ↓
requirements/*_lock.txt
    │
    ↓
MODULE.bazel (pip.parse)
    │
    ↓
@pip//package_name
    │
    ↓
py_library/py_binary (deps = ["@pip//package"])
```

### JavaScript Integration

```
package.json
    │
    ↓
pnpm install
    │
    ↓
pnpm-lock.yaml
    │
    ↓
MODULE.bazel (npm.pnpm_lock)
    │
    ↓
//:node_modules/package_name
    │
    ↓
js_library (deps = ["//:node_modules/package"])
```

### Go Integration

```
import "github.com/example/pkg"
    │
    ↓
go get / go mod tidy
    │
    ↓
go.mod + go.sum
    │
    ↓
bazel mod tidy
    │
    ↓
MODULE.bazel (use_repo)
    │
    ↓
@com_github_example_pkg
    │
    ↓
go_library (deps = ["@com_github_example_pkg"])
```

### Rust Integration

```
Cargo.toml
    │
    ↓
cargo add / cargo update
    │
    ↓
Cargo.lock
    │
    ↓
MODULE.bazel (crates.from_cargo)
    │
    ↓
@crates//:package_name
    │
    ↓
rust_library (deps = ["@crates//:package"])
```

## Extension Points

Blueprint provides several extension points:

### 1. Custom BUILD Rules

Add custom Starlark rules in repository:

```starlark
# //tools/rules/my_rule.bzl
def my_custom_rule(name, ...):
    # Custom build logic
    pass
```

### 2. Aspect CLI Plugins

Create `.star` files in `.aspect/cli/`:

```starlark
# .aspect/cli/my_plugin.star
def declare_targets(ctx):
    # Analyze source files
    # Generate targets
    pass
```

### 3. Gazelle Extensions

Add custom language support or BUILD generation logic.

### 4. Custom Linters

Add linters in `tools/lint/`:

```starlark
# tools/lint/BUILD
lint_aspect(
    name = "my_linter",
    binary = "//path/to:linter",
    configs = ["//:.linter.config"],
)
```

### 5. Pre-commit Hooks

Add hooks in `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: local
    hooks:
      - id: my-hook
        name: My Custom Hook
        entry: ./tools/my-hook.sh
        language: script
```

### 6. Bazel Configuration

Add custom `.bazelrc` flags:

```bash
# .bazelrc
build:myconfig --flag=value
```

## Data Flow

### Development Workflow

```
1. Developer writes code
    ↓
2. Gazelle generates BUILD files (optional)
    ↓
3. Developer runs `bazel build //path:target`
    ↓
4. Bazel analyzes dependencies
    ↓
5. Bazel executes actions (compile, link, etc.)
    ↓
6. Outputs cached
    ↓
7. Binary ready to run
```

### CI/CD Workflow

```
1. Code pushed to repository
    ↓
2. CI checks out code
    ↓
3. CI runs `bazel test //...`
    ↓
4. Bazel uses remote cache (if configured)
    ↓
5. Tests run in parallel
    ↓
6. Results reported
    ↓
7. Artifacts published (on success)
```

## Performance Characteristics

### Caching Layers

1. **In-Memory Cache** - Active Bazel process
2. **Local Disk Cache** - Persistent across builds
3. **Remote Cache** - Shared across team/CI
4. **Repository Cache** - Downloaded dependencies

### Incremental Builds

Bazel rebuilds only:
- Changed files
- Files that depend on changed files
- Transitively affected targets

### Parallelism

Bazel executes:
- Independent actions in parallel
- Limited by `--jobs` flag
- Can use remote execution for massive parallelism

## Security Considerations

- **Sandboxed Execution** - Actions run in isolated sandbox
- **Reproducible Builds** - Same inputs = same outputs
- **Dependency Pinning** - Lock files ensure exact versions
- **Supply Chain** - All dependencies explicitly declared

## Next Steps

- Review [Contributing Guide](contributing.md)
- Explore [Development Setup](development.md)
- Read [Testing Guide](testing.md)
