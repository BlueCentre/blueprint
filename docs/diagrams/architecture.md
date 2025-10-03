# Architecture Diagrams

This document provides visual representations of Blueprint's architecture and workflows.

## Table of Contents

- [System Overview](#system-overview)
- [Build System Architecture](#build-system-architecture)
- [Developer Workflow](#developer-workflow)
- [Dependency Management](#dependency-management)
- [Linting and Formatting](#linting-and-formatting)
- [CI/CD Pipeline](#cicd-pipeline)

## System Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Blueprint System                             │
└─────────────────────────────────────────────────────────────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
        ▼                         ▼                         ▼
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│   Developer  │         │     Build    │         │     CI/CD    │
│  Workstation │────────▶│    System    │◀────────│   Pipeline   │
└──────────────┘         │   (Bazel)    │         └──────────────┘
                         └──────────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    │             │             │
                    ▼             ▼             ▼
            ┌────────────┐ ┌────────────┐ ┌────────────┐
            │  Language  │ │   Tools    │ │   Cache    │
            │ Toolchains │ │   & Utils  │ │  (Local/   │
            └────────────┘ └────────────┘ │  Remote)   │
                                          └────────────┘
```

### Technology Stack

```
┌─────────────────────────────────────────────────────────┐
│                    User Interface                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │   CLI    │  │   IDE    │  │ direnv   │             │
│  └──────────┘  └──────────┘  └──────────┘             │
└─────────────────────────────────────────────────────────┘
                        ▼
┌─────────────────────────────────────────────────────────┐
│              Developer Experience Layer                  │
│  ┌────────────────┐  ┌────────────────┐                │
│  │  Aspect CLI    │  │  bazel_env.bzl │                │
│  │  - build       │  │  - Tool mgmt   │                │
│  │  - lint        │  │  - PATH setup  │                │
│  │  - configure   │  └────────────────┘                │
│  └────────────────┘                                     │
│  ┌────────────────┐  ┌────────────────┐                │
│  │  Gazelle       │  │  rules_lint    │                │
│  │  - Code gen    │  │  - Linting     │                │
│  │  - BUILD files │  │  - Formatting  │                │
│  └────────────────┘  └────────────────┘                │
└─────────────────────────────────────────────────────────┘
                        ▼
┌─────────────────────────────────────────────────────────┐
│                 Bazel Build System                       │
│  ┌────────────────────────────────────────────────┐    │
│  │              bzlmod (MODULE.bazel)              │    │
│  │  - Dependency resolution                        │    │
│  │  - Module graph                                 │    │
│  │  - Toolchain registration                       │    │
│  └────────────────────────────────────────────────┘    │
│  ┌────────────────────────────────────────────────┐    │
│  │            Build Graph & Execution              │    │
│  │  - Action graph                                 │    │
│  │  - Dependency analysis                          │    │
│  │  - Sandboxed execution                          │    │
│  │  - Caching (local & remote)                     │    │
│  └────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
                        ▼
┌─────────────────────────────────────────────────────────┐
│                Language Ecosystems                       │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐         │
│  │Python│ │  Go  │ │  JS  │ │ Rust │ │ Java │  etc.   │
│  │ pip  │ │go.mod│ │ pnpm │ │Cargo │ │Maven │         │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘         │
└─────────────────────────────────────────────────────────┘
```

## Build System Architecture

### Bazel Build Process

```
    ┌──────────────┐
    │ Source Files │
    └──────┬───────┘
           │
           ▼
    ┌──────────────┐
    │ BUILD Files  │◀───────┐
    └──────┬───────┘        │
           │           ┌────┴─────┐
           ▼           │ Gazelle  │
    ┌──────────────┐  │(Auto-gen)│
    │   Loading    │  └──────────┘
    │   Phase      │
    └──────┬───────┘
           │  Parse Starlark
           │  Load dependencies
           │
           ▼
    ┌──────────────┐
    │  Analysis    │
    │   Phase      │
    └──────┬───────┘
           │  Build dependency graph
           │  Apply aspects (linting)
           │  Determine actions
           │
           ▼
    ┌──────────────┐
    │  Execution   │
    │   Phase      │
    └──────┬───────┘
           │
           ├─────────┐
           │         │
           ▼         ▼
    ┌──────────┐ ┌──────────┐
    │  Cache   │ │  Action  │
    │  Lookup  │ │Execution │
    └────┬─────┘ └─────┬────┘
         │             │
         │  Cache Hit  │ Cache Miss
         └──────┬──────┘
                │
                ▼
         ┌──────────────┐
         │   Outputs    │
         │   (Cached)   │
         └──────────────┘
```

### Module Dependency Resolution

```
MODULE.bazel
    │
    ├─── Direct Dependencies
    │    │
    │    ├─── rules_python@1.6.3
    │    │    └─── pip extension
    │    │         └─── Parse requirements
    │    │              └─── @pip//package
    │    │
    │    ├─── aspect_rules_js@2.6.0
    │    │    └─── pnpm extension
    │    │         └─── Parse pnpm-lock.yaml
    │    │              └─── //:node_modules/package
    │    │
    │    ├─── rules_go@0.57.0
    │    │    └─── go_deps extension
    │    │         └─── Parse go.mod
    │    │              └─── @com_github_org_repo
    │    │
    │    └─── rules_rust@0.63.0
    │         └─── crates extension
    │              └─── Parse Cargo.toml
    │                   └─── @crates//:package
    │
    └─── Transitive Dependencies
         (Automatically resolved by Bazel)
```

## Developer Workflow

### Code → Build → Test Flow

```
┌──────────────────────────────────────────────────────────────┐
│  Step 1: Write Code                                           │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐             │
│  │  Python    │  │     Go     │  │ TypeScript │             │
│  │  .py files │  │  .go files │  │  .ts files │             │
│  └────────────┘  └────────────┘  └────────────┘             │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│  Step 2: Generate BUILD Files (Optional)                     │
│  $ bazel run //:gazelle                                      │
│  ┌──────────────────────────────────────┐                   │
│  │ Gazelle analyzes imports and         │                   │
│  │ generates/updates BUILD files        │                   │
│  └──────────────────────────────────────┘                   │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│  Step 3: Format Code                                         │
│  $ format                                                    │
│  ┌──────────────────────────────────────┐                   │
│  │ - Python: ruff format                │                   │
│  │ - JavaScript: prettier               │                   │
│  │ - Go: gofmt                          │                   │
│  └──────────────────────────────────────┘                   │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│  Step 4: Lint Code                                           │
│  $ aspect lint //...                                         │
│  ┌──────────────────────────────────────┐                   │
│  │ rules_lint applies language-specific │                   │
│  │ linters via Bazel aspects            │                   │
│  └──────────────────────────────────────┘                   │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│  Step 5: Build                                               │
│  $ bazel build //path/to:target                             │
│  ┌──────────────────────────────────────┐                   │
│  │ Bazel compiles code, resolves deps,  │                   │
│  │ and caches outputs                   │                   │
│  └──────────────────────────────────────┘                   │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│  Step 6: Test                                                │
│  $ bazel test //path/to:test                                │
│  ┌──────────────────────────────────────┐                   │
│  │ Bazel runs tests in sandbox,         │                   │
│  │ caches results                       │                   │
│  └──────────────────────────────────────┘                   │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│  Step 7: Run                                                 │
│  $ bazel run //path/to:binary                               │
│  ┌──────────────────────────────────────┐                   │
│  │ Execute the built binary             │                   │
│  └──────────────────────────────────────┘                   │
└──────────────────────────────────────────────────────────────┘
```

### Tool Management Flow

```
    ┌─────────────────────┐
    │ tools.lock.json     │
    │ - Tool: copier      │
    │ - URL: github.com/..│
    │ - Version: v1.2.3   │
    └──────────┬──────────┘
               │
               ▼
    ┌─────────────────────┐
    │ bazel run           │
    │ //tools:bazel_env   │
    └──────────┬──────────┘
               │
               │  Downloads & Installs
               │
               ▼
    ┌─────────────────────────────────────┐
    │ bazel-out/bazel_env-opt/bin/        │
    │   tools/bazel_env/bin/              │
    │   ├── copier                        │
    │   ├── yq                            │
    │   ├── pnpm                          │
    │   └── gazelle -> ../../gazelle/...  │
    └────────────┬────────────────────────┘
                 │
                 ▼
    ┌─────────────────────┐
    │ .envrc              │
    │ PATH_add bazel-out/.│
    └──────────┬──────────┘
               │
               ▼
    ┌─────────────────────┐
    │ direnv allow        │
    └──────────┬──────────┘
               │
               ▼
    ┌─────────────────────┐
    │ Tools on PATH       │
    │ $ copier --version  │
    │ $ yq --version      │
    └─────────────────────┘
```

## Dependency Management

### Python Dependency Flow

```
1. Developer Action:
   Edit pyproject.toml
   
      dependencies = [
        "requests>=2.28.0"
      ]
           │
           ▼
2. Update Lock Files:
   ./tools/repin
   
      (runs uv pip compile)
           │
           ▼
3. Generated Files:
   requirements/
     requirements_lock.txt
     
      requests==2.31.0
      certifi==2023.7.22
      ...
           │
           ▼
4. Bazel MODULE:
   MODULE.bazel
   
      pip.parse(
        requirements_lock = 
          "//requirements:requirements_lock.txt"
      )
           │
           ▼
5. Available in BUILD:
   
      deps = [
        "@pip//requests",
      ]
```

### JavaScript Dependency Flow

```
1. Developer Action:
   pnpm add package-name
           │
           ▼
2. Updated Files:
   package.json
   pnpm-lock.yaml
           │
           ▼
3. Bazel MODULE:
   MODULE.bazel
   
      npm.pnpm_lock(
        pnpm_lock = "//:pnpm-lock.yaml"
      )
           │
           ▼
4. Link All Packages:
   BUILD
   
      npm_link_all_packages(
        name = "node_modules"
      )
           │
           ▼
5. Available in BUILD:
   
      deps = [
        "//:node_modules/package-name",
      ]
```

### Go Dependency Flow

```
1. Developer Action:
   go get github.com/example/pkg
           │
           ▼
2. Update go.mod:
   go mod tidy
           │
           ▼
3. Updated Files:
   go.mod
   go.sum
           │
           ▼
4. Update MODULE.bazel:
   bazel mod tidy
   
      use_repo(go_deps,
        "com_github_example_pkg"
      )
           │
           ▼
5. Update BUILD Files:
   bazel run //:gazelle
           │
           ▼
6. Available in BUILD:
   
      deps = [
        "@com_github_example_pkg",
      ]
```

## Linting and Formatting

### rules_lint Architecture

```
    ┌──────────────┐
    │ Source Files │
    └──────┬───────┘
           │
           │  bazel lint //...
           │
           ▼
    ┌──────────────────────────┐
    │   Bazel Aspects          │
    │   (rules_lint)           │
    └──────┬───────────────────┘
           │
           ├───────────┬───────────┬───────────┐
           ▼           ▼           ▼           ▼
      ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐
      │  ruff  │  │ eslint │  │  nogo  │  │shellchk│
      │(Python)│  │  (JS)  │  │  (Go)  │  │(Shell) │
      └───┬────┘  └───┬────┘  └───┬────┘  └───┬────┘
          │           │           │           │
          ▼           ▼           ▼           ▼
      ┌───────────────────────────────────────────┐
      │         Report Files                      │
      │  bazel-out/.../ruff.json                 │
      │  bazel-out/.../eslint.json               │
      │  bazel-out/.../nogo.json                 │
      └──────────────┬────────────────────────────┘
                     │
                     ▼
      ┌──────────────────────────┐
      │  Aspect CLI Collects     │
      │  - Aggregates reports    │
      │  - Formats output        │
      │  - Interactive fixes     │
      └──────────────────────────┘
```

### Format Command Flow

```
    $ format path/to/file.py
           │
           ▼
    ┌──────────────────┐
    │ Detect Language  │
    └────────┬─────────┘
             │
    ┌────────┴────────┐
    │                 │
    ▼                 ▼
┌─────────┐      ┌─────────┐
│ Python  │      │   JS    │
│  ruff   │      │prettier │
│ format  │      │ --write │
└─────────┘      └─────────┘
```

## CI/CD Pipeline

### GitHub Actions Workflow

```
┌──────────────────────────────────────────────────────────┐
│  Trigger: Push / Pull Request                             │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────────┐
│  Checkout Code                                            │
│  - actions/checkout@v4                                   │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────────┐
│  Setup Bazel                                              │
│  - bazel-contrib/setup-bazel@0.9.0                       │
│  - Reads .bazelversion                                   │
└────────────────────┬─────────────────────────────────────┘
                     │
           ┌─────────┴─────────┐
           │                   │
           ▼                   ▼
┌──────────────────┐  ┌──────────────────┐
│  Build           │  │  Lint            │
│  bazel build //..│  │  aspect lint //..│
└────────┬─────────┘  └────────┬─────────┘
         │                     │
         └──────────┬──────────┘
                    │
                    ▼
┌──────────────────────────────────────────────────────────┐
│  Test                                                     │
│  bazel test //...                                        │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────────┐
│  Upload Artifacts (optional)                              │
│  - Test reports                                          │
│  - Coverage reports                                      │
│  - Build outputs                                         │
└──────────────────────────────────────────────────────────┘
```

### Remote Cache Integration

```
    ┌──────────────┐
    │  Developer   │
    │  Workstation │
    └──────┬───────┘
           │
           │  bazel build --remote_cache=...
           │
           ▼
    ┌──────────────────┐
    │  Remote Cache    │
    │  (Shared Storage)│
    │                  │
    │  ┌────────────┐  │
    │  │ Artifacts  │  │
    │  │ ├─ lib.so  │  │
    │  │ ├─ app     │  │
    │  │ └─ test    │  │
    │  └────────────┘  │
    └────────┬─────────┘
             │
             │  Cache hits/misses
             │
             ▼
    ┌──────────────┐
    │   CI/CD      │
    │   Pipeline   │
    └──────────────┘
```

## Data Flow Summary

```
Source Code
    │
    ├─→ Gazelle → BUILD Files
    │
    ├─→ Bazel → Build Artifacts
    │
    ├─→ rules_lint → Lint Reports
    │
    └─→ formatters → Formatted Code

Dependencies
    │
    ├─→ pip/pnpm/go/cargo → Lock Files
    │
    └─→ MODULE.bazel → Bazel Targets

Developer
    │
    ├─→ direnv → Tools on PATH
    │
    ├─→ bazel build → Outputs
    │
    └─→ bazel test → Test Results
```

## Component Interaction Matrix

| Component | Interacts With | Purpose |
|-----------|----------------|---------|
| Bazel | All languages, cache, tools | Build orchestration |
| Gazelle | Source files, BUILD files | Code generation |
| rules_lint | Linters, source files | Quality checks |
| bazel_env | Tools, direnv, PATH | Tool management |
| Aspect CLI | Bazel, developers | Enhanced UX |
| direnv | bazel_env, shell | Environment setup |

## Next Steps

- Review [Architecture Overview](../contributor/architecture.md) for detailed explanations
- Explore [Developer Workflows](../user/workflows.md) for practical examples
- Check [Contributing Guide](../contributor/contributing.md) to get involved
