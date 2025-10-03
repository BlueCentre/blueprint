# Polyglot Bazel Starter


    # This is executable Markdown that's tested on CI.
    set -o errexit -o nounset -o xtrace
    alias ~~~=":<<'~~~sh'";:<<'~~~sh'

**Blueprint** is a production-ready polyglot Bazel starter repository that provides a foundation for building multi-language projects with best practices, optimized configurations, and excellent developer experience.

## Features

- 🧱 **Latest Bazel** - Up-to-date Bazel version with optimized configuration
- 🌐 **Multi-Language Support** - Python, Go, JavaScript/TypeScript, Rust, Java, Kotlin, C++
- 📦 **Curated Configuration** - Optimized `.bazelrc` flags via [bazelrc-preset.bzl]
- 🧰 **Developer Tools** - Automated environment setup with [bazel_env.bzl]
- 🎨 **Code Quality** - Formatting and linting using rules_lint
- ✅ **Pre-commit Hooks** - Automatic code formatting and validation
- 🚀 **Aspect Workflows** - Enhanced Bazel developer experience
- 📚 **Comprehensive Docs** - Detailed documentation for users, contributors, and admins

## Quick Start

### 1. Clone and Setup

```bash
git clone https://github.com/BlueCentre/blueprint.git
cd blueprint

# Allow direnv to set up environment
direnv allow

# If needed, run bazel_env setup
bazel run //tools:bazel_env
direnv allow
```

### 2. Build and Test

```bash
# Build everything
bazel build //...

# Run tests
bazel test //...

# Format code
format

# Lint code
aspect lint //...
```

### 3. Explore the Documentation

📖 **[Full Documentation →](docs/README.md)**

## Documentation

Comprehensive documentation organized for different audiences:

### 📚 For Users

New to Blueprint? Start here:

- **[Getting Started](docs/user/getting-started.md)** - Quick start guide
- **[Project Structure](docs/user/project-structure.md)** - Understanding the repository layout
- **[Developer Workflows](docs/user/workflows.md)** - Common development tasks
- **[Language Guides](docs/languages/README.md)** - Python, Go, JS, Rust, Java, C++, Shell
- **[Troubleshooting](docs/user/troubleshooting.md)** - Common issues and solutions
- **[FAQ](docs/user/faq.md)** - Frequently asked questions

### 🤝 For Contributors

Want to contribute? Check these guides:

- **[Contributing Guide](docs/contributor/contributing.md)** - How to contribute
- **[Architecture Overview](docs/contributor/architecture.md)** - System design
- **[Development Setup](docs/contributor/development.md)** - Setting up your dev environment
- **[Testing Guide](docs/contributor/testing.md)** - Writing and running tests
- **[Code Style Guide](docs/contributor/code-style.md)** - Coding standards

### 🔧 For Administrators

Maintaining Blueprint:

- **[Maintenance Guide](docs/admin/maintenance.md)** - Regular maintenance tasks
- **[CI/CD Configuration](docs/admin/ci-cd.md)** - Continuous integration setup
- **[Release Process](docs/admin/releases.md)** - Creating releases
- **[Security Guide](docs/admin/security.md)** - Security best practices

### 📊 Visual Documentation

- **[Architecture Diagrams](docs/diagrams/architecture.md)** - System architecture visualizations

## Supported Languages

Blueprint provides first-class support for:

| Language | Package Manager | Documentation |
|----------|----------------|---------------|
| **Python** | pip/uv | [Guide](docs/languages/python.md) |
| **Go** | go modules | [Guide](docs/languages/go.md) |
| **JavaScript/TypeScript** | pnpm | [Guide](docs/languages/javascript.md) |
| **Rust** | Cargo | [Guide](docs/languages/rust.md) |
| **Java/Kotlin** | Maven | [Guide](docs/languages/java-kotlin.md) |
| **C/C++** | System | [Guide](docs/languages/cpp.md) |
| **Shell** | N/A | [Guide](docs/languages/shell.md) |

## Developer environment

> Before following these instructions, setup the developer environment by running <code>direnv allow</code> and follow any prompts.
> This ensures that tools we call in the following steps will be on the PATH.

Many commands are available on the PATH thanks to direnv:

~~~sh
copier --help
yq --help
~~~
