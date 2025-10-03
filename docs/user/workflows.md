# Developer Workflows

This guide covers common development tasks and workflows when working with Blueprint.

## Table of Contents

- [Building](#building)
- [Testing](#testing)
- [Formatting and Linting](#formatting-and-linting)
- [Managing Dependencies](#managing-dependencies)
- [Working with Containers](#working-with-containers)
- [Debugging](#debugging)
- [Release Builds](#release-builds)

## Building

### Build Everything

```bash
# Build all targets in the workspace
bazel build //...
```

### Build Specific Targets

```bash
# Build a single target
bazel build //path/to:target

# Build multiple targets
bazel build //path/to:target1 //path/to:target2

# Build all targets in a package
bazel build //path/to:all
```

### Build with Options

```bash
# Build in debug mode
bazel build -c dbg //path/to:target

# Build in optimized mode
bazel build -c opt //path/to:target

# Build with specific platform
bazel build --platforms=//tools/platforms:linux_amd64 //path/to:target
```

### Understanding Build Output

```bash
# Show build commands
bazel build //path/to:target --subcommands

# Explain why target was rebuilt
bazel build //path/to:target --explain=explain.txt

# Show execution log
bazel build //path/to:target --execution_log_json_file=exec.json
```

## Testing

### Run All Tests

```bash
# Run all tests in the workspace
bazel test //...
```

### Run Specific Tests

```bash
# Run tests in a package
bazel test //path/to:all

# Run a single test
bazel test //path/to:test_target

# Run tests matching a pattern
bazel test //... --test_tag_filters=unit
```

### Test Options

```bash
# Run tests with verbose output
bazel test //path/to:test --test_output=all

# Run tests and show errors only
bazel test //path/to:test --test_output=errors

# Run flaky tests multiple times
bazel test //path/to:test --runs_per_test=3

# Run tests with specific arguments
bazel test //path/to:test --test_arg=--verbose
```

### Test Logs

```bash
# View test logs
cat bazel-testlogs/path/to/test/test.log

# Find test logs
find bazel-testlogs -name "*.log"
```

## Formatting and Linting

### Code Formatting

Blueprint uses multiple formatters managed through `rules_lint`:

```bash
# Format all files
format

# Format specific files
format path/to/file.py path/to/file.ts

# Check formatting without modifying files
bazel test //tools/format:format_check
```

### Pre-commit Hooks

```bash
# Install pre-commit hooks
pre-commit install

# Run pre-commit on all files
pre-commit run --all-files

# Run pre-commit on staged files
pre-commit run
```

### Linting

```bash
# Lint all targets
aspect lint //...

# Lint specific targets
aspect lint //path/to:target

# Lint with autofix
aspect lint --fix //...

# Show lint rules
aspect lint --help
```

**Linters by language:**

- **Python:** Ruff
- **JavaScript/TypeScript:** ESLint
- **Go:** nogo (staticcheck)
- **Shell:** shellcheck
- **C/C++:** clang-tidy
- **Java:** PMD
- **Kotlin:** ktlint

## Managing Dependencies

### Python Dependencies

```bash
# Add a dependency to pyproject.toml
vim pyproject.toml  # Add package to [project.dependencies]

# Update lock files
./tools/repin

# Update Gazelle-generated BUILD files
bazel run //:gazelle
```

For console scripts:

```bash
# Create a py_console_script_binary
cat << 'EOF' | buildozer -f -
new_load @rules_python//python/entry_points:py_console_script_binary.bzl py_console_script_binary
new py_console_script_binary scriptname
set pkg "@pip//package_name"
EOF
```

### JavaScript/TypeScript Dependencies

```bash
# Add a package
pnpm add package-name

# Add a dev dependency
pnpm add -D package-name

# Remove a package
pnpm remove package-name

# Update all packages
pnpm update

# Install dependencies
pnpm install
```

### Go Dependencies

```bash
# Add a dependency (adds to go.mod)
go get github.com/example/package

# Update go.mod and go.sum
go mod tidy -v

# Update MODULE.bazel
bazel mod tidy

# Update BUILD files
bazel run //:gazelle
```

### Rust Dependencies

```bash
# Add a dependency
cargo add package-name

# Add a dev dependency
cargo add --dev package-name

# Update dependencies
cargo update

# Update BUILD files (if using rules_rust with cargo-bazel)
bazel run //path:crates_vendor
```

### Java/Kotlin Dependencies

Edit `MODULE.bazel` and add to `maven.install()`:

```starlark
maven.install(
    artifacts = [
        "io.grpc:grpc-all:1.51.1",
        "com.google.guava:guava:31.1-jre",  # Add here
    ],
)
```

Then update the lock file:

```bash
bazel run @unpinned_maven//:pin
```

## Working with Containers

### Building OCI Images

Blueprint supports building container images with special pragmas:

**Python image:**

```python
# File: __main__.py
# oci: build

def main():
    print("Hello from container!")

if __name__ == "__main__":
    main()
```

```bash
# Build the image
bazel build //path/to:image

# Load image into Docker
bazel run //path/to:image
```

**Go image:**

```go
// File: main.go
// oci: build

package main

func main() {
    println("Hello from container!")
}
```

```bash
# Build the image
bazel build //path/to:image
```

### Local Kubernetes Development with kind

The dev container includes **kind** (Kubernetes in Docker) for local cluster development. This is optimized for resource-constrained environments.

#### Managing the Cluster

```bash
# Create a new cluster (single node, resource-efficient)
./tools/kind-cluster.sh create

# Check cluster status
./tools/kind-cluster.sh status

# Restart cluster
./tools/kind-cluster.sh restart

# Delete cluster to free resources
./tools/kind-cluster.sh delete
```

#### Deploying to kind

**Load local images into kind:**
```bash
# Build your image
bazel build //path/to:image

# Load into Docker
bazel run //path/to:image

# Load into kind cluster
kind load docker-image your-image:tag --name blueprint-dev
```

**Deploy with kubectl:**
```bash
# Create deployment
kubectl create deployment my-app --image=your-image:tag

# Expose service
kubectl expose deployment my-app --port=8080 --target-port=8080

# Port forward to access locally
kubectl port-forward deployment/my-app 8080:8080
```

**Using Skaffold for continuous development:**
```bash
# Initialize skaffold configuration
skaffold init

# Continuous development mode (auto-rebuild on changes)
skaffold dev

# Deploy once
skaffold run
```

#### Accessing Services

The kind cluster is configured with port mappings:
- `localhost:8080` → cluster port 80 (HTTP)
- `localhost:8443` → cluster port 443 (HTTPS)

**Example Ingress:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
spec:
  rules:
  - host: localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app
            port:
              number: 8080
```

After applying this Ingress, access your app at `http://localhost:8080`

#### Resource Management

**Monitor cluster resources:**
```bash
# Check node resources
kubectl top nodes

# Check pod resources
kubectl top pods

# View resource usage
kubectl describe node blueprint-dev-control-plane
```

**Clean up unused resources:**
```bash
# Delete unused pods/services
kubectl delete deployment,service --all

# Or delete the entire cluster when not in use
./tools/kind-cluster.sh delete
```

## Debugging

### Debug Bazel

```bash
# Show detailed build information
bazel build //path/to:target -s

# Explain build decisions
bazel cquery //path/to:target --output=graph

# Analyze action graph
bazel aquery //path/to:target

# Run with debug output
bazel build //path/to:target --sandbox_debug
```

### Debug Tests

```bash
# Run test with debugger
bazel test //path/to:test --run_under=gdb

# Run test with custom tool
bazel test //path/to:test --run_under=/path/to/tool

# Keep test outputs
bazel test //path/to:test --test_output=all --cache_test_results=no
```

### Clean Build

```bash
# Clean build outputs
bazel clean

# Deep clean (removes all cached artifacts)
bazel clean --expunge

# Clean specific targets
bazel clean //path/to:target
```

## Release Builds

### Stamping

Blueprint supports stamping builds with version information:

```bash
# Build with stamping enabled
bazel build --config=release //path/to:target
```

Available stamp variables (from `tools/workspace_status.sh`):

- `STABLE_GIT_COMMIT` - Current commit hash
- `STABLE_MONOREPO_VERSION` - Semver-compatible version

Use in `expand_template` or similar rules:

```starlark
load("@aspect_bazel_lib//lib:expand_template.bzl", "expand_template")

expand_template(
    name = "version",
    template = "version.txt.in",
    out = "version.txt",
    stamp_substitutions = {
        "{{VERSION}}": "{{STABLE_MONOREPO_VERSION}}",
        "{{COMMIT}}": "{{STABLE_GIT_COMMIT}}",
    },
)
```

### Creating Releases

```bash
# Build release artifacts
bazel build --config=release //...

# Run release tests
bazel test --config=release //...

# Publish artifacts (example)
bazel run --config=release //path/to:publish
```

## Running Binaries

### Run with Bazel

```bash
# Run a binary target
bazel run //path/to:binary

# Run with arguments
bazel run //path/to:binary -- --arg1 --arg2

# Run with environment variables
bazel run //path/to:binary --action_env=VAR=value
```

### Run Generated Binary

```bash
# Build first
bazel build //path/to:binary

# Run directly
./bazel-bin/path/to/binary
```

## Development Tools

### Available Commands

Thanks to bazel_env.bzl and direnv, many tools are on PATH:

```bash
# Copy project templates
copier --help

# YAML processing
yq --help

# Package manager
pnpm --help

# Gazelle (BUILD file generator)
bazel run //:gazelle

# Update bazel_env
bazel run //tools:bazel_env
```

### Adding New Tools

1. Add tool to `tools/tools.lock.json`
2. Run `bazel run //tools:bazel_env`
3. Run `direnv allow`
4. Tool is now available on PATH

See the [Admin Guide](../admin/maintenance.md#managing-tools) for details.

## Performance Tips

### Build Performance

```bash
# Use remote cache (if configured)
bazel build //... --remote_cache=...

# Build with more parallelism
bazel build //... --jobs=16

# Use Aspect CLI for better performance
aspect build //...
```

### Disk Space

```bash
# Clean old outputs
bazel clean

# Remove old repositories
bazel clean --expunge

# Check disk usage
bazel info output_base
du -sh $(bazel info output_base)
```

## Next Steps

- Explore [Language-Specific Guides](../languages/README.md)
- Learn about [Troubleshooting](troubleshooting.md)
- Understand [Architecture](../contributor/architecture.md)
