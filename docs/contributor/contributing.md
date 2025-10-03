# Contributing to Blueprint

Thank you for your interest in contributing to Blueprint! This guide will help you get started.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Contribution Guidelines](#contribution-guidelines)
- [Code Review Process](#code-review-process)
- [Testing Requirements](#testing-requirements)
- [Documentation](#documentation)

## Code of Conduct

We are committed to providing a welcoming and inclusive environment. Please:

- Be respectful and constructive
- Welcome newcomers and help them learn
- Focus on what is best for the community
- Show empathy towards other community members

## Getting Started

### 1. Fork and Clone

```bash
# Fork on GitHub, then clone
git clone https://github.com/YOUR_USERNAME/blueprint.git
cd blueprint

# Add upstream remote
git remote add upstream https://github.com/BlueCentre/blueprint.git
```

### 2. Set Up Development Environment

```bash
# Allow direnv
direnv allow

# Run bazel_env setup
bazel run //tools:bazel_env
direnv allow

# Install pre-commit hooks
pre-commit install
```

### 3. Create a Branch

```bash
# Update from upstream
git fetch upstream
git checkout main
git merge upstream/main

# Create feature branch
git checkout -b feature/your-feature-name
```

## Development Workflow

### Making Changes

1. **Write your code**
   - Follow existing code patterns
   - Add tests for new functionality
   - Update documentation

2. **Format code**
   ```bash
   format
   ```

3. **Run linters**
   ```bash
   aspect lint //...
   ```

4. **Run tests**
   ```bash
   bazel test //...
   ```

5. **Build everything**
   ```bash
   bazel build //...
   ```

### Commit Guidelines

Follow conventional commits format:

```
type(scope): subject

body

footer
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Formatting, no code change
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

**Examples:**

```bash
git commit -m "feat(python): add support for Python 3.12"
git commit -m "fix(gazelle): handle edge case in BUILD generation"
git commit -m "docs(readme): update installation instructions"
```

### Keeping Your Branch Updated

```bash
# Fetch upstream changes
git fetch upstream

# Rebase your branch
git rebase upstream/main

# Resolve conflicts if any
# Then continue
git rebase --continue
```

## Contribution Guidelines

### What to Contribute

We welcome contributions in these areas:

✅ **Bug fixes** - Fix issues in existing code
✅ **Features** - New functionality that benefits users
✅ **Documentation** - Improve or add documentation
✅ **Tests** - Add or improve test coverage
✅ **Performance** - Optimize build times or runtime
✅ **Examples** - Add example projects or use cases

### What Not to Contribute

❌ **Breaking changes** without discussion
❌ **Personal preferences** without clear benefit
❌ **Large refactors** without prior agreement
❌ **Unrelated changes** in same PR
❌ **Generated files** (like BUILD files) without source changes

### Before Starting Major Work

For significant changes:

1. **Open an issue** to discuss the change
2. **Wait for feedback** from maintainers
3. **Create design doc** if needed
4. **Get approval** before implementing

This prevents wasted effort on changes that won't be accepted.

## Code Review Process

### Submitting a Pull Request

1. **Push your branch**
   ```bash
   git push origin feature/your-feature-name
   ```

2. **Open Pull Request** on GitHub
   - Use descriptive title
   - Fill out PR template
   - Link related issues
   - Add screenshots if UI changes

3. **Respond to feedback**
   - Address review comments
   - Push additional commits
   - Request re-review when ready

### Review Criteria

Reviewers check for:

- ✅ **Correctness** - Does it work as intended?
- ✅ **Tests** - Are there adequate tests?
- ✅ **Documentation** - Is it documented?
- ✅ **Code quality** - Is it maintainable?
- ✅ **Performance** - Are there performance concerns?
- ✅ **Compatibility** - Does it break existing features?

### Getting Your PR Merged

Requirements for merge:

1. **Passing CI** - All checks must pass
2. **Approval** - At least one maintainer approval
3. **No conflicts** - Branch must be up-to-date
4. **All feedback addressed** - No unresolved comments

## Testing Requirements

### Unit Tests

All new code must have unit tests:

```starlark
py_test(
    name = "test_feature",
    srcs = ["test_feature.py"],
    deps = [":feature"],
)
```

### Integration Tests

For features that integrate multiple components:

```starlark
sh_test(
    name = "integration_test",
    srcs = ["integration_test.sh"],
    data = [
        ":app",
        "testdata",
    ],
)
```

### Test Coverage

- Aim for **>80% coverage** for new code
- Test edge cases and error conditions
- Include positive and negative test cases

### Running Tests Locally

```bash
# Run all tests
bazel test //...

# Run specific tests
bazel test //path/to:test

# Run with coverage
bazel coverage //...

# Run tests multiple times (for flaky detection)
bazel test //... --runs_per_test=10
```

## Documentation

### What to Document

1. **User-facing features** - How to use new functionality
2. **API changes** - Document new APIs or changes
3. **Breaking changes** - Clearly mark and document
4. **Examples** - Show how to use features
5. **Configuration** - Document new config options

### Where to Document

- **docs/** - User guides, tutorials, references
- **README.md** - High-level overview and quick start
- **Code comments** - Complex logic or non-obvious code
- **BUILD files** - Target descriptions
- **CHANGELOG.md** - Keep track of changes

### Documentation Style

- Use clear, concise language
- Include code examples
- Add links to related documentation
- Use proper markdown formatting
- Test commands and examples

## Project Structure for Contributors

Key directories for contributors:

```
blueprint/
├── .aspect/cli/        # Aspect CLI extensions
├── docs/               # Documentation (you are here)
├── tools/              # Build and dev tools
│   ├── format/        # Formatting config
│   ├── lint/          # Linting config
│   └── platforms/     # Platform definitions
├── MODULE.bazel        # Bazel dependencies
├── BUILD               # Root BUILD file
└── .github/           # GitHub Actions CI
```

## Development Tips

### Faster Development

```bash
# Use Aspect CLI for better performance
aspect build //...

# Build with disk cache
bazel build --disk_cache=~/.bazel/cache //...

# Run only affected tests
bazel test //path/to/changed:all
```

### Debugging

```bash
# Show build commands
bazel build //path:target -s

# Show why target rebuilt
bazel build //path:target --explain=explain.txt

# Verbose logging
bazel build //path:target --verbose_failures --subcommands
```

### Cleaning

```bash
# Clean build outputs
bazel clean

# Deep clean (removes all caches)
bazel clean --expunge
```

## Release Process

(For maintainers)

1. Update version in relevant files
2. Update CHANGELOG.md
3. Create git tag: `git tag -a v1.2.3 -m "Release v1.2.3"`
4. Push tag: `git push origin v1.2.3`
5. GitHub Actions creates release

## Getting Help

### Resources

- **Documentation**: Read the [docs](../README.md)
- **Architecture**: Understand the [architecture](architecture.md)
- **Issues**: Check [existing issues](https://github.com/BlueCentre/blueprint/issues)
- **Discussions**: Join [discussions](https://github.com/BlueCentre/blueprint/discussions)

### Ask Questions

- Open a [GitHub Discussion](https://github.com/BlueCentre/blueprint/discussions)
- Join [Bazel Slack](https://slack.bazel.build/)
- Ask on [Stack Overflow](https://stackoverflow.com/questions/tagged/bazel)

### Report Bugs

Use the [issue template](https://github.com/BlueCentre/blueprint/issues/new) and include:

- Blueprint version
- Bazel version
- Operating system
- Steps to reproduce
- Expected vs actual behavior
- Relevant logs or error messages

## Recognition

Contributors are recognized in:

- **CONTRIBUTORS.md** - List of all contributors
- **Release notes** - Credit for features and fixes
- **GitHub insights** - Contribution graphs

Thank you for contributing to Blueprint! 🎉

## Next Steps

- Review [Architecture Documentation](architecture.md)
- Check [Development Setup Guide](development.md)
- Read [Testing Guide](testing.md)
- Explore [Code Style Guide](code-style.md)
