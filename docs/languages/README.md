# Language-Specific Guides

Blueprint supports multiple programming languages. This page provides links to detailed guides for each supported language.

## Supported Languages

### Core Languages

- **[Python](python.md)** - Full support with pip/uv package management
- **[Go](go.md)** - Full support with go modules
- **[JavaScript/TypeScript](javascript.md)** - Full support with pnpm

### Additional Languages

- **[Rust](rust.md)** - Full support with Cargo
- **[Java/Kotlin](java-kotlin.md)** - Full support with Maven
- **[C/C++](cpp.md)** - Full support with native toolchains
- **[Shell](shell.md)** - Support for bash and sh scripts

## Quick Reference

| Language | Package Manager | Add Dependency | Generate BUILD |
|----------|----------------|----------------|----------------|
| Python | pip/uv | Edit `pyproject.toml`, run `./tools/repin` | `bazel run //:gazelle` |
| Go | go modules | `go get pkg`, `go mod tidy`, `bazel mod tidy` | `bazel run //:gazelle` |
| JavaScript/TypeScript | pnpm | `pnpm add package` | Automatic |
| Rust | Cargo | `cargo add package` | `bazel run //path:crates_vendor` |
| Java/Kotlin | Maven | Edit `MODULE.bazel`, run `@maven//:pin` | Manual |
| C/C++ | N/A | System libraries | Manual |
| Shell | N/A | N/A | Automatic |

## Language Rules

Each language is supported through Bazel rules:

- **Python**: `@aspect_rules_py` (Aspect's enhanced Python rules)
- **Go**: `@rules_go` (Official Go rules)
- **JavaScript**: `@aspect_rules_js` (Aspect's JavaScript rules)
- **TypeScript**: `@aspect_rules_ts` (Aspect's TypeScript rules)
- **Rust**: `@rules_rust` (Official Rust rules)
- **Java**: `@rules_java` (Official Java rules)
- **Kotlin**: `@rules_kotlin` (Official Kotlin rules)
- **C/C++**: `@rules_cc` (Official C++ rules)
- **Shell**: `@rules_shell` (Shell script rules)

## Multi-Language Projects

Blueprint excels at multi-language projects. You can:

1. **Mix languages in the same repository**
   ```
   my-app/
   ├── backend/       # Go services
   ├── frontend/      # TypeScript UI
   ├── ml-model/      # Python models
   └── shared/        # Shared protos/configs
   ```

2. **Share code between languages**
   ```starlark
   # Protocol buffers
   proto_library(
       name = "api_proto",
       srcs = ["api.proto"],
   )
   
   # Python bindings
   py_proto_library(
       name = "api_py_proto",
       deps = [":api_proto"],
   )
   
   # Go bindings
   go_proto_library(
       name = "api_go_proto",
       protos = [":api_proto"],
   )
   ```

3. **Use consistent tooling**
   - Single build system (Bazel)
   - Unified linting (`aspect lint`)
   - Common formatting (`format`)
   - Shared CI/CD

## Language Selection Guide

### When to Use Python

✅ Good for:
- Data science and ML
- Scripting and automation
- Web APIs (Django, FastAPI)
- Rapid prototyping

❌ Consider alternatives for:
- High-performance computing
- System-level programming
- Mobile applications

### When to Use Go

✅ Good for:
- Microservices and APIs
- Cloud-native applications
- CLI tools
- System programming
- Concurrent systems

❌ Consider alternatives for:
- CPU-intensive tasks
- Frontend development
- Data science

### When to Use JavaScript/TypeScript

✅ Good for:
- Web frontend
- Node.js backends
- Full-stack applications
- Real-time applications

❌ Consider alternatives for:
- CPU-intensive tasks
- System programming
- High-performance computing

### When to Use Rust

✅ Good for:
- System programming
- Performance-critical code
- WebAssembly
- Embedded systems
- Safe concurrent programming

❌ Consider alternatives for:
- Rapid prototyping
- Simple scripts
- Steep learning curve

### When to Use Java/Kotlin

✅ Good for:
- Enterprise applications
- Android development (Kotlin)
- Large-scale systems
- JVM ecosystem integration

❌ Consider alternatives for:
- Resource-constrained environments
- Quick scripts
- System programming

### When to Use C/C++

✅ Good for:
- System programming
- Performance-critical code
- Hardware interaction
- Legacy integration

❌ Consider alternatives for:
- Rapid development
- Memory safety concerns
- Modern web applications

## Getting Started with Each Language

### 1. Choose Your Language

Pick the language(s) that best fit your project requirements.

### 2. Review Language Guide

Read the detailed guide for setup and best practices.

### 3. Set Up Dependencies

Configure package manager and add required dependencies.

### 4. Write Code

Follow language conventions and Bazel best practices.

### 5. Generate BUILD Files

Use Gazelle or create BUILD files manually.

### 6. Build and Test

```bash
bazel build //path/to:target
bazel test //path/to:test
```

## Cross-Language Integration

### Protocol Buffers

Share data structures across languages:

```protobuf
// api.proto
syntax = "proto3";

message Request {
  string id = 1;
  string name = 2;
}
```

Generate bindings for each language:

```starlark
proto_library(name = "api_proto", srcs = ["api.proto"])
py_proto_library(name = "api_py_proto", deps = [":api_proto"])
go_proto_library(name = "api_go_proto", protos = [":api_proto"])
```

### Foreign Function Interface (FFI)

Call between languages:

- **Python ↔ C/C++**: Use ctypes or pybind11
- **Go ↔ C**: Use cgo
- **Rust ↔ C**: Native FFI support
- **JavaScript ↔ Native**: Use N-API or WebAssembly

### Microservices

Different languages for different services:

```
services/
├── api-gateway/        # Go
├── user-service/       # Python
├── payment-service/    # Java
└── notification/       # Node.js
```

## Language-Specific Tools

Each language has specialized tools available:

### Python
- **Linter**: ruff
- **Formatter**: ruff format
- **Type Checker**: mypy (can add)
- **Test Runner**: pytest

### Go
- **Linter**: nogo (staticcheck)
- **Formatter**: gofmt
- **Test Runner**: go test (via rules_go)

### JavaScript/TypeScript
- **Linter**: eslint
- **Formatter**: prettier
- **Type Checker**: tsc
- **Test Runner**: jest, vitest

### Rust
- **Linter**: clippy
- **Formatter**: rustfmt
- **Test Runner**: cargo test (via rules_rust)

### Java/Kotlin
- **Linter**: PMD (Java), ktlint (Kotlin)
- **Formatter**: google-java-format
- **Test Runner**: JUnit

## Best Practices

1. **Use language idioms** - Follow language-specific conventions
2. **Leverage Bazel** - Use Bazel's features (caching, sandboxing)
3. **Generate BUILD files** - Use Gazelle when possible
4. **Test thoroughly** - Write tests for all languages
5. **Document clearly** - Explain multi-language interactions
6. **Keep dependencies minimal** - Reduce build complexity

## Resources

### Official Documentation
- [Bazel Language Rules](https://bazel.build/rules)
- [Aspect Rules](https://docs.aspect.build/rulesets/)

### Language-Specific
- [Python Rules Docs](https://rules-python.readthedocs.io/)
- [Go Rules Docs](https://github.com/bazelbuild/rules_go)
- [JavaScript Rules Docs](https://docs.aspect.build/rulesets/aspect_rules_js/)
- [Rust Rules Docs](https://bazelbuild.github.io/rules_rust/)

### Community
- [Bazel Slack](https://slack.bazel.build/)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/bazel)

## Next Steps

Choose a language and dive into its specific guide:

- [Python Guide](python.md) →
- [Go Guide](go.md) →
- [JavaScript/TypeScript Guide](javascript.md) →
- [Rust Guide](rust.md) →
- [Java/Kotlin Guide](java-kotlin.md) →
- [C/C++ Guide](cpp.md) →
- [Shell Guide](shell.md) →
