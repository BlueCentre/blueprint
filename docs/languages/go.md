# Go Guide

This guide covers working with Go in Blueprint using `rules_go`.

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

Go projects in Blueprint use:
- **Go toolchain** (version managed by Bazel via go.mod)
- **go.mod** for dependency management
- **Gazelle** for BUILD file generation

### Initial Configuration

Blueprint is pre-configured for Go with:

```starlark
# MODULE.bazel
bazel_dep(name = "rules_go", version = "0.57.0")
bazel_dep(name = "gazelle", version = "0.45.0")

go_sdk = use_extension("@rules_go//go:extensions.bzl", "go_sdk")
go_sdk.from_file(go_mod = "//:go.mod")
```

## Project Structure

Typical Go package structure:

```
my-go-package/
├── BUILD
├── main.go
├── lib.go
├── lib_test.go
└── README.md
```

### BUILD File Example

```starlark
load("@rules_go//go:def.bzl", "go_library", "go_binary", "go_test")

go_library(
    name = "mylib",
    srcs = [
        "lib.go",
    ],
    importpath = "github.com/example/project/my-go-package",
    visibility = ["//visibility:public"],
    deps = [
        "@com_github_spf13_cobra//:cobra",
        "@org_golang_x_sync//errgroup",
    ],
)

go_binary(
    name = "app",
    srcs = ["main.go"],
    embed = [":mylib"],
)

go_test(
    name = "mylib_test",
    srcs = ["lib_test.go"],
    embed = [":mylib"],
    deps = [
        "@com_github_stretchr_testify//assert",
    ],
)
```

## Dependencies

### Adding Dependencies

1. **Add import to Go code:**

```go
import "github.com/spf13/cobra"
```

2. **Update go.mod:**

```bash
go get github.com/spf13/cobra
# or
go mod tidy
```

3. **Update MODULE.bazel:**

```bash
bazel mod tidy
```

This adds the dependency to `use_repo()` in MODULE.bazel.

4. **Update BUILD files:**

```bash
bazel run //:gazelle
```

### Using Dependencies

Reference dependencies in BUILD files:

```starlark
go_library(
    name = "mylib",
    srcs = ["lib.go"],
    importpath = "github.com/example/project/mylib",
    deps = [
        "@com_github_spf13_cobra//:cobra",      # External
        "//other/package:lib",                   # Internal
    ],
)
```

### Gazelle Import Path Mapping

Gazelle automatically maps Go import paths to Bazel labels based on `go.mod`.

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
bazel run //path/to:app -- --flag value

# With environment variables
bazel run //path/to:app --action_env=DEBUG=1
```

### Build for Different Platforms

```bash
# Build for Linux AMD64
bazel build --platforms=@rules_go//go/toolchain:linux_amd64 //path/to:app

# Build for macOS ARM64
bazel build --platforms=@rules_go//go/toolchain:darwin_arm64 //path/to:app

# Build for Windows
bazel build --platforms=@rules_go//go/toolchain:windows_amd64 //path/to:app
```

## Testing

### Writing Tests

Use standard Go testing:

```go
// lib_test.go
package mypackage

import (
    "testing"
    "github.com/stretchr/testify/assert"
)

func TestFunction(t *testing.T) {
    result := MyFunction()
    assert.Equal(t, expected, result)
}

func TestTableDriven(t *testing.T) {
    tests := []struct {
        name     string
        input    int
        expected int
    }{
        {"case1", 1, 2},
        {"case2", 2, 4},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Double(tt.input)
            assert.Equal(t, tt.expected, result)
        })
    }
}
```

### Running Tests

```bash
# Run all Go tests
bazel test //path/to:all

# Run specific test
bazel test //path/to:mylib_test

# With verbose output
bazel test //path/to:mylib_test --test_output=all

# With test flags
bazel test //path/to:mylib_test --test_arg=-test.v
```

### Test Data

Include test data files:

```starlark
go_test(
    name = "mylib_test",
    srcs = ["lib_test.go"],
    data = [
        "testdata/input.json",
        "testdata/expected.txt",
    ],
    embed = [":mylib"],
)
```

Access in test code:

```go
import "github.com/bazelbuild/rules_go/go/runfiles"

r, _ := runfiles.New()
path, _ := r.Rlocation("_main/path/to/testdata/input.json")
```

## Linting and Formatting

### gofmt

Go code is automatically formatted:

```bash
# Format all Go files
format

# Or use gofmt directly
gofmt -w .
```

### nogo (Static Analysis)

Blueprint uses nogo for static analysis (includes staticcheck):

```starlark
# MODULE.bazel
go_sdk.nogo(nogo = "//tools/lint:nogo")
```

Configuration in `tools/lint/BUILD`:

```starlark
nogo(
    name = "nogo",
    visibility = ["//visibility:public"],
    deps = [
        "@org_golang_x_tools//go/analysis/passes/...",
    ],
)
```

### Linting

```bash
# Lint all Go targets
aspect lint //...

# Lint specific target
aspect lint //path/to:mylib
```

## Advanced Topics

### CGO

For C integration:

```go
// #include <stdio.h>
// void hello() { printf("Hello from C\n"); }
import "C"

func CallC() {
    C.hello()
}
```

```starlark
go_library(
    name = "cgo_lib",
    srcs = ["cgo.go"],
    cgo = True,
    cdeps = ["//third_party/c:lib"],
)
```

### Protocol Buffers

Generate Go code from protos:

```starlark
load("@rules_proto//proto:defs.bzl", "proto_library")
load("@rules_go//proto:def.bzl", "go_proto_library")

proto_library(
    name = "api_proto",
    srcs = ["api.proto"],
)

go_proto_library(
    name = "api_go_proto",
    importpath = "github.com/example/project/api",
    proto = ":api_proto",
)
```

Use in code:

```go
import pb "github.com/example/project/api"

msg := &pb.Request{
    Id: "123",
    Name: "example",
}
```

### Docker/OCI Images

Blueprint supports building Go container images with pragmas:

```go
// main.go
// oci: build

package main

func main() {
    println("Hello from container!")
}
```

This automatically generates an `image` target:

```bash
# Build image
bazel build //path/to:image

# Load into Docker
bazel run //path/to:image
```

### Workspaces and Modules

Use go workspaces for local development:

```bash
# Create workspace
go work init
go work use .
go work use ./submodule

# Bazel still uses go.mod for dependencies
```

### Embed Directive

Embed files in binaries:

```go
//go:embed templates/*
var templates embed.FS

func main() {
    content, _ := templates.ReadFile("templates/index.html")
    fmt.Println(string(content))
}
```

```starlark
go_binary(
    name = "app",
    srcs = ["main.go"],
    embedsrcs = glob(["templates/*"]),
)
```

### Generated Code

For code generation tools like stringer:

```starlark
load("@rules_go//go:def.bzl", "go_library")

# Use genrule or custom rules
genrule(
    name = "gen_stringer",
    srcs = ["types.go"],
    outs = ["types_string.go"],
    cmd = "$(location @org_golang_x_tools//cmd/stringer) -type=Type $< > $@",
    tools = ["@org_golang_x_tools//cmd/stringer"],
)

go_library(
    name = "mylib",
    srcs = [
        "types.go",
        ":gen_stringer",
    ],
    importpath = "github.com/example/project/mylib",
)
```

## Common Patterns

### CLI Application

```go
// main.go
package main

import (
    "flag"
    "fmt"
)

func main() {
    var name string
    flag.StringVar(&name, "name", "World", "Name to greet")
    flag.Parse()
    
    fmt.Printf("Hello, %s!\n", name)
}
```

```starlark
go_binary(
    name = "cli",
    srcs = ["main.go"],
)
```

Run:
```bash
bazel run //path/to:cli -- --name=Blueprint
```

### HTTP Server

```go
// server.go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

```starlark
go_binary(
    name = "server",
    srcs = ["server.go"],
)
```

### Microservice

```go
// service.go
package main

import (
    "context"
    "log"
    "net"
    
    "google.golang.org/grpc"
    pb "github.com/example/project/api"
)

type server struct {
    pb.UnimplementedServiceServer
}

func (s *server) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.User, error) {
    return &pb.User{Id: req.Id, Name: "John"}, nil
}

func main() {
    lis, _ := net.Listen("tcp", ":50051")
    s := grpc.NewServer()
    pb.RegisterServiceServer(s, &server{})
    log.Fatal(s.Serve(lis))
}
```

## Troubleshooting

### Module Not Found

```bash
# Update go.mod
go mod tidy

# Update MODULE.bazel
bazel mod tidy

# Regenerate BUILD files
bazel run //:gazelle
```

### Import Errors

Check that:
1. Package is in `go.mod`
2. MODULE.bazel has the dependency in `use_repo()`
3. BUILD file has the dependency in `deps`

### Gazelle Not Updating

```bash
# Update gazelle directives in root BUILD
# gazelle:prefix github.com/example/project

# Force update
bazel run //:gazelle -- update-repos -from_file=go.mod
bazel run //:gazelle
```

### CGO Errors

Ensure:
- C dependencies are available
- `cgo = True` in go_library
- Correct `cdeps` specified

## Best Practices

1. **Use go.mod** - Manage dependencies with go modules
2. **Keep imports organized** - Group standard, external, and internal
3. **Write tests** - Aim for high coverage
4. **Use interfaces** - For better testability and flexibility
5. **Follow conventions** - Use standard Go project layout
6. **Document packages** - Write package-level documentation
7. **Use contexts** - For cancellation and timeouts
8. **Handle errors** - Don't ignore errors

## Resources

- [rules_go Documentation](https://github.com/bazelbuild/rules_go)
- [Gazelle Documentation](https://github.com/bazelbuild/bazel-gazelle)
- [Go by Example](https://gobyexample.com/)
- [Effective Go](https://go.dev/doc/effective_go)

## Next Steps

- Explore [Testing Guide](../contributor/testing.md)
- Learn about [Docker Images](../user/workflows.md#working-with-containers)
- Check [Troubleshooting](../user/troubleshooting.md#go-module-not-found)
