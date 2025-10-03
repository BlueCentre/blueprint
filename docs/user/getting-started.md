
# Getting Started with Blueprint

This guide will help you get started with Blueprint, a polyglot Bazel starter repository.

## Prerequisites

Before you begin, ensure you have the following installed:

- **Git** - Version control system
- **Bazel** or **Bazelisk** - Build tool (Bazelisk recommended for automatic version management)
- **direnv** - Environment variable manager for tool setup
- **Node.js** and **pnpm** - For JavaScript/TypeScript projects (optional)
- **Python 3** - For Python projects (optional)
- **Go** - For Go projects (optional)
- **Rust and Cargo** - For Rust projects (optional)
- **Java JDK** - For Java/Kotlin projects (optional)

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/BlueCentre/blueprint.git
cd blueprint
```

### 2. Set Up Developer Environment

Blueprint uses direnv to automatically configure your development environment:

```bash
# Allow direnv to load environment variables
direnv allow
```

This command will:
- Add Bazel-managed tools to your PATH
- Set up language-specific environments
- Configure shell completions

**Important:** If you see an error message, run the following to initialize the environment:

```bash
bazel run //tools:bazel_env
direnv allow
```

### 3. Verify Installation

Check that everything is set up correctly:

```bash
# Verify Bazel is working
bazel version

# Check that tools are available
copier --help
yq --help
```

### 4. (Optional) Set Up Kubernetes for Container Development

If you're working with containerized applications, set up a local Kubernetes cluster using kind:

```bash
# Verify Docker is running
docker info

# Create a local Kubernetes cluster
./tools/kind-cluster.sh create

# Check cluster status
./tools/kind-cluster.sh status

# Verify kubectl access
kubectl get nodes
```

The kind cluster includes:
- Single-node setup optimized for limited resources
- Port mappings: `localhost:8080` → cluster port 80, `localhost:8443` → cluster port 443
- Automatic kubectl configuration

**Cluster management commands:**
```bash
./tools/kind-cluster.sh create    # Create cluster
./tools/kind-cluster.sh status    # Check status
./tools/kind-cluster.sh restart   # Restart cluster
./tools/kind-cluster.sh delete    # Remove cluster
```

**Note:** The cluster is not started automatically. Create it when needed and delete it to free resources.

### 5. Build the Project

```bash
# Build everything
bazel build //...

# Or build a specific target
bazel build //path/to:target
```

### 5. Run Tests

```bash
# Run all tests
bazel test //...

# Run specific tests
bazel test //path/to:test
```

## Next Steps

### Explore the Codebase

Familiarize yourself with the project structure:

- **`MODULE.bazel`** - Bazel module dependencies
- **`BUILD`** files - Build target definitions
- **`.bazelrc`** - Bazel configuration flags
- **`tools/`** - Build and development tools
- **`.aspect/cli/`** - Aspect CLI configuration and extensions

See [Project Structure](project-structure.md) for details.

### Choose Your Language

Blueprint supports multiple languages. See the language-specific guides:

- [Python Guide](../languages/python.md)
- [Go Guide](../languages/go.md)
- [JavaScript/TypeScript Guide](../languages/javascript.md)
- [Rust Guide](../languages/rust.md)
- [Java/Kotlin Guide](../languages/java-kotlin.md)
- [C/C++ Guide](../languages/cpp.md)

### Common Development Tasks

Learn about common workflows:

- **Code Formatting:** Run `format` to format all files
- **Linting:** Run `aspect lint //...` to check for issues
- **Adding Dependencies:** See language-specific guides
- **Running Targets:** Use `bazel run //path/to:target`

See [Developer Workflows](workflows.md) for more details.

## Using Blueprint as a Template

If you want to create a new project based on Blueprint:

### Option 1: Use GitHub Template

1. Click "Use this template" on the GitHub repository
2. Create your new repository
3. Clone and set up as described above

### Option 2: Fork and Customize

1. Fork the repository
2. Clone your fork
3. Update project metadata:
   - `package.json` - Update name and metadata
   - `pyproject.toml` - Update project name
   - `go.mod` - Update module path
   - `Cargo.toml` - Update package name
   - `BUILD` files - Update gazelle prefix

### Option 3: Copy Files

```bash
# Create new project
mkdir my-project
cd my-project

# Copy Blueprint files
cp -r /path/to/blueprint/{.aspect,.bazelrc,.bazelversion,MODULE.bazel,BUILD,tools} .

# Initialize git
git init
git add .
git commit -m "Initial commit from Blueprint"
```

## Understanding Bazel Concepts

If you're new to Bazel, here are key concepts:

- **Workspace** - The root directory containing your source code
- **Package** - A directory containing a BUILD file
- **Target** - A buildable unit (library, binary, test, etc.)
- **Label** - A reference to a target (e.g., `//path/to/package:target`)
- **Rule** - A build instruction (e.g., `py_library`, `go_binary`)
- **Aspect** - A way to augment build rules (used for linting)

## Getting Help

If you run into issues:

1. Check the [Troubleshooting Guide](troubleshooting.md)
2. Review the [FAQ](faq.md)
3. Search existing [GitHub Issues](https://github.com/BlueCentre/blueprint/issues)
4. Read [Bazel documentation](https://bazel.build/docs)
5. Consult [Aspect Workflows docs](https://docs.aspect.build/)

## What's Next?

- Learn about [Project Structure](project-structure.md)
- Explore [Developer Workflows](workflows.md)
- Deep dive into [Architecture](../contributor/architecture.md)
- Understand [Contributing Guidelines](../contributor/contributing.md)
