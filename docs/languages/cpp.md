# C/C++ Guide

This guide covers working with C/C++ in Blueprint using `rules_cc`.

## Quick Start

### Project Structure

```
my-cpp-package/
├── BUILD
├── lib.h
├── lib.cc
└── main.cc
```

### BUILD File

```starlark
load("@rules_cc//cc:defs.bzl", "cc_library", "cc_binary", "cc_test")

cc_library(
    name = "mylib",
    srcs = ["lib.cc"],
    hdrs = ["lib.h"],
    visibility = ["//visibility:public"],
)

cc_binary(
    name = "app",
    srcs = ["main.cc"],
    deps = [":mylib"],
)

cc_test(
    name = "test",
    srcs = ["test.cc"],
    deps = [
        ":mylib",
        "@com_google_googletest//:gtest_main",
    ],
)
```

## Example Code

```cpp
// lib.h
#pragma once
#include <string>

namespace myapp {
    std::string greet(const std::string& name);
}

// lib.cc
#include "lib.h"

namespace myapp {
    std::string greet(const std::string& name) {
        return "Hello, " + name + "!";
    }
}

// main.cc
#include <iostream>
#include "lib.h"

int main() {
    std::cout << myapp::greet("Blueprint") << std::endl;
    return 0;
}
```

## Building and Testing

```bash
# Build
bazel build //path/to:app

# Run
bazel run //path/to:app

# Test
bazel test //path/to:test

# With optimization
bazel build -c opt //path/to:app

# With debug symbols
bazel build -c dbg //path/to:app
```

## Compiler Options

```starlark
cc_binary(
    name = "app",
    srcs = ["main.cc"],
    copts = [
        "-std=c++17",
        "-Wall",
        "-Wextra",
    ],
    linkopts = ["-lpthread"],
    deps = [":mylib"],
)
```

## Resources

- [rules_cc Documentation](https://github.com/bazelbuild/rules_cc)
- [Bazel C++ Tutorial](https://bazel.build/tutorials/cpp)
