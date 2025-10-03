# Rust Guide

This guide covers working with Rust in Blueprint using `rules_rust`.

## Quick Start

### Project Structure

```
my-rust-package/
├── BUILD
├── Cargo.toml
├── src/
│   ├── lib.rs
│   └── main.rs
└── tests/
    └── integration_test.rs
```

### BUILD File

```starlark
load("@rules_rust//rust:defs.bzl", "rust_library", "rust_binary", "rust_test")

rust_library(
    name = "mylib",
    srcs = ["src/lib.rs"],
    edition = "2021",
    deps = [
        "@crates//:serde",
        "@crates//:serde_json",
    ],
)

rust_binary(
    name = "app",
    srcs = ["src/main.rs"],
    edition = "2021",
    deps = [":mylib"],
)

rust_test(
    name = "mylib_test",
    crate = ":mylib",
)
```

## Adding Dependencies

1. **Add to Cargo.toml:**
   ```bash
   cargo add serde serde_json
   ```

2. **Use in code:**
   ```rust
   use serde::{Serialize, Deserialize};
   
   #[derive(Serialize, Deserialize)]
   struct Data {
       name: String,
   }
   ```

3. **Build:**
   ```bash
   bazel build //path/to:app
   ```

## Building and Testing

```bash
# Build
bazel build //path/to:app

# Run
bazel run //path/to:app

# Test
bazel test //path/to:mylib_test

# Format
format  # Uses rustfmt

# Lint
cargo clippy
```

## Resources

- [rules_rust Documentation](https://bazelbuild.github.io/rules_rust/)
- [Rust Book](https://doc.rust-lang.org/book/)
