# Java/Kotlin Guide

This guide covers working with Java and Kotlin in Blueprint.

## Quick Start

### Java Project Structure

```
my-java-package/
├── BUILD
└── src/
    └── main/
        └── java/
            └── com/
                └── example/
                    └── App.java
```

### Kotlin Project Structure

```
my-kotlin-package/
├── BUILD
└── src/
    └── main/
        └── kotlin/
            └── com/
                └── example/
                    └── App.kt
```

## Java BUILD File

```starlark
load("@rules_java//java:defs.bzl", "java_library", "java_binary", "java_test")

java_library(
    name = "mylib",
    srcs = glob(["src/main/java/**/*.java"]),
    deps = [
        "@maven//:com_google_guava_guava",
    ],
)

java_binary(
    name = "app",
    main_class = "com.example.App",
    runtime_deps = [":mylib"],
)

java_test(
    name = "test",
    srcs = glob(["src/test/java/**/*.java"]),
    test_class = "com.example.AppTest",
    deps = [
        ":mylib",
        "@maven//:junit_junit",
    ],
)
```

## Kotlin BUILD File

```starlark
load("@rules_kotlin//kotlin:jvm.bzl", "kt_jvm_library", "kt_jvm_binary")

kt_jvm_library(
    name = "mylib",
    srcs = glob(["src/main/kotlin/**/*.kt"]),
    deps = [
        "@maven//:org_jetbrains_kotlinx_kotlinx_coroutines_core",
    ],
)

kt_jvm_binary(
    name = "app",
    main_class = "com.example.AppKt",
    runtime_deps = [":mylib"],
)
```

## Adding Maven Dependencies

Edit `MODULE.bazel`:

```starlark
maven.install(
    artifacts = [
        "com.google.guava:guava:31.1-jre",
        "junit:junit:4.13.2",
    ],
)
```

Then update:

```bash
bazel run @maven//:pin
```

## Building and Testing

```bash
# Build
bazel build //path/to:app

# Run
bazel run //path/to:app

# Test
bazel test //path/to:test

# Format (Java)
google-java-format -i src/**/*.java

# Lint
bazel build //... --aspects //tools/lint:linters.bzl%pmd
```

## Resources

- [rules_java Documentation](https://github.com/bazelbuild/rules_java)
- [rules_kotlin Documentation](https://github.com/bazelbuild/rules_kotlin)
