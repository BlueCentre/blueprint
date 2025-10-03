# Contributing to Blueprint

Thank you for your interest in contributing to Blueprint!

## Quick Links

📖 **Full Contributing Guide:** [docs/contributor/contributing.md](docs/contributor/contributing.md)

This file contains comprehensive information about:
- Code of conduct
- Getting started with development
- Development workflow
- Code review process
- Testing requirements
- Documentation guidelines

## Quick Start for Contributors

1. **Read the documentation:**
   - [Contributing Guide](docs/contributor/contributing.md)
   - [Development Setup](docs/contributor/development.md)
   - [Code Style Guide](docs/contributor/code-style.md)

2. **Set up your environment:**
   ```bash
   git clone https://github.com/YOUR_USERNAME/blueprint.git
   cd blueprint
   direnv allow
   bazel run //tools:bazel_env
   ```

3. **Make your changes:**
   ```bash
   git checkout -b feature/my-feature
   # Make changes
   format
   aspect lint //...
   bazel test //...
   ```

4. **Submit a pull request**

## Getting Help

- **Documentation:** [docs/README.md](docs/README.md)
- **Issues:** [GitHub Issues](https://github.com/BlueCentre/blueprint/issues)
- **Discussions:** [GitHub Discussions](https://github.com/BlueCentre/blueprint/discussions)

## Code of Conduct

We are committed to providing a welcoming and inclusive environment. Please be respectful and constructive in all interactions.

## License

By contributing to Blueprint, you agree that your contributions will be licensed under the same license as the project.
