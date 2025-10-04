# Development Setup Guide

This guide helps contributors set up their development environment for working on Blueprint.

## Prerequisites

### Required Tools

- **Git** - Version control
- **Bazelisk** - Bazel version manager (recommended over Bazel)
- **direnv** - Environment variable manager
- **Code editor** - VS Code, IntelliJ, or your preferred editor

### Optional Tools

- **Docker** - For container development
- **pre-commit** - For git hooks

## Initial Setup

### 1. Fork and Clone

```bash
# Fork on GitHub, then clone your fork
git clone https://github.com/YOUR_USERNAME/blueprint.git
cd blueprint

# Add upstream remote
git remote add upstream https://github.com/BlueCentre/blueprint.git
```

### 2. Install Bazelisk

**macOS:**
```bash
brew install bazelisk
```

**Linux:**
```bash
wget https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64
chmod +x bazelisk-linux-amd64
sudo mv bazelisk-linux-amd64 /usr/local/bin/bazel
```

**Windows:**
```powershell
choco install bazelisk
```

### 3. Install direnv

**macOS:**
```bash
brew install direnv
echo 'eval "$(direnv hook bash)"' >> ~/.bashrc
# or for zsh
echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc
```

**Linux:**
```bash
curl -sfL https://direnv.net/install.sh | bash
echo 'eval "$(direnv hook bash)"' >> ~/.bashrc
```

### 4. Allow direnv

```bash
direnv allow
```

If you see an error, run:

```bash
bazel run //tools:bazel_env
direnv allow
```

### 5. Install pre-commit

```bash
pip install pre-commit
pre-commit install
```

## IDE Setup

### VS Code

1. **Install extensions:**
   - Bazel (bazel-stack-vscode.bazel-stack-vscode)
   - Python (ms-python.python)
   - Go (golang.go)
   - ESLint (dbaeumer.vscode-eslint)
   - Prettier (esbenp.prettier-vscode)

2. **Configure settings:**

`.vscode/settings.json` (already in repository):

```json
{
  "bazel.buildifierExecutable": "${workspaceFolder}/bazel-bin/tools/buildifier",
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

### IntelliJ IDEA

1. **Install Bazel plugin:**
   - Preferences → Plugins → Search "Bazel" → Install

2. **Import project:**
   - File → Import Bazel Project
   - Select workspace root
   - Use existing `.bazelproject` file

### Vim/Neovim

Add to your config:

```vim
" LSP support for Bazel
Plug 'neovim/nvim-lspconfig'

" Bazel syntax
Plug 'bazelbuild/vim-bazel'
```

## Development Workflow

### Making Changes

```bash
# Create a branch
git checkout -b feature/my-feature

# Make changes
# ...

# Format code
format

# Lint code
aspect lint //...

# Run tests
bazel test //...

# Commit
git commit -m "feat: add new feature"
```

### Keeping Your Fork Updated

```bash
# Fetch upstream changes
git fetch upstream

# Update main
git checkout main
git merge upstream/main

# Rebase your branch
git checkout feature/my-feature
git rebase main
```

## Testing Your Changes

### Run All Tests

```bash
bazel test //...
```

### Run Specific Tests

```bash
# Test specific package
bazel test //path/to:all

# Test single target
bazel test //path/to:test_name

# With output
bazel test //path/to:test_name --test_output=all
```

### Run Tests Multiple Times

For flaky test detection:

```bash
bazel test //path/to:test --runs_per_test=10
```

## Building Documentation

```bash
# Documentation is in Markdown
# Preview with any Markdown viewer

# Or use a local server
uv pip install grip
uv run grip
# Visit http://localhost:6419/docs/
```

## Debugging

### Debug Build Issues

```bash
# Show commands
bazel build //path/to:target -s

# Explain rebuild
bazel build //path/to:target --explain=explain.txt

# Verbose failures
bazel build //path/to:target --verbose_failures
```

### Debug Test Failures

```bash
# Run with all output
bazel test //path/to:test --test_output=all

# Run with debugger
bazel test //path/to:test --run_under=gdb

# Keep test outputs
bazel test //path/to:test --cache_test_results=no
```

### Clean Builds

```bash
# Clean outputs
bazel clean

# Deep clean
bazel clean --expunge
```

## Common Tasks

### Add a New Language Support

1. Add rules to `MODULE.bazel`
2. Configure toolchain
3. Add linter configuration
4. Update documentation
5. Add examples

### Update Dependencies

**Bazel:**
```bash
# Update .bazelversion
echo "8.4.1" > .bazelversion
```

**Python:**
```bash
# Edit pyproject.toml
./tools/repin
bazel run //:gazelle_python_manifest.update
bazel run //:gazelle
```

**JavaScript:**
```bash
pnpm update
```

**Go:**
```bash
go get -u ./...
go mod tidy
bazel mod tidy
bazel run //:gazelle
```

### Add New Tool

This project uses [rules_multitool](https://github.com/theoremlp/rules_multitool) to manage cross-platform CLI tools. Tools are defined in `tools/tools.lock.json` and automatically made available via `bazel run //tools:bazel_env`.

#### Steps to Add a Tool

1. **Find the latest version and download URLs**

   For most tools, check their GitHub releases page or official download site. You'll need:
   - Latest version number
   - Download URLs for all platforms (Linux x86_64, Linux arm64, macOS x86_64, macOS arm64)
   - Binary type: single file or archive (tar.gz, zip)

   Example for kubectl:
   ```bash
   # Get latest version
   curl -sL https://api.github.com/repos/kubernetes/kubernetes/releases/latest | grep tag_name
   
   # Version URLs follow pattern:
   # https://dl.k8s.io/release/v1.34.1/bin/linux/amd64/kubectl
   # https://dl.k8s.io/release/v1.34.1/bin/linux/arm64/kubectl
   # https://dl.k8s.io/release/v1.34.1/bin/darwin/amd64/kubectl
   # https://dl.k8s.io/release/v1.34.1/bin/darwin/arm64/kubectl
   ```

2. **Compute SHA256 checksums**

   Download each binary/archive and compute its SHA256 hash:
   ```bash
   # For direct binary files
   curl -sL https://dl.k8s.io/release/v1.34.1/bin/linux/amd64/kubectl | sha256sum
   
   # For archives - hash the archive file itself, not the extracted binary
   curl -sL https://url/to/archive.tar.gz -o archive.tar.gz && sha256sum archive.tar.gz
   ```

   **Important:** For archives, you must hash the downloaded archive file, not the extracted binary.

3. **Add tool definition to `tools/tools.lock.json`**

   Add a new entry with platform-specific binaries. Two formats are supported:

   **For single binary files:**
   ```json
   "kubectl": {
     "binaries": [
       {
         "kind": "file",
         "url": "https://dl.k8s.io/release/v1.34.1/bin/linux/amd64/kubectl",
         "sha256": "7721f265e18709862655affba5343e85e1980639395d5754473dafaadcaa69e3",
         "os": "linux",
         "cpu": "x86_64"
       },
       {
         "kind": "file",
         "url": "https://dl.k8s.io/release/v1.34.1/bin/linux/arm64/kubectl",
         "sha256": "420e6110e3ba7ee5a3927b5af868d18df17aae36b720529ffa4e9e945aa95450",
         "os": "linux",
         "cpu": "arm64"
       },
       {
         "kind": "file",
         "url": "https://dl.k8s.io/release/v1.34.1/bin/darwin/amd64/kubectl",
         "sha256": "bb211f2b31f2b3bc60562b44cc1e3b712a16a98e9072968ba255beb04cefcfdf",
         "os": "macos",
         "cpu": "x86_64"
       },
       {
         "kind": "file",
         "url": "https://dl.k8s.io/release/v1.34.1/bin/darwin/arm64/kubectl",
         "sha256": "d80e5fa36f2b14005e5bb35d3a72818acb1aea9a081af05340a000e5fbdb2f76",
         "os": "macos",
         "cpu": "arm64"
       }
     ]
   }
   ```

   **For archives (tar.gz, zip):**
   ```json
   "gcloud": {
     "binaries": [
       {
         "kind": "archive",
         "url": "https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-linux-x86_64.tar.gz",
         "file": "google-cloud-sdk/bin/gcloud",
         "sha256": "1bd66d2f4bf45ebfcfdc98b109c262ad022af9d8ce9ebad8afa06faa48620e0e",
         "os": "linux",
         "cpu": "x86_64"
       },
       // ... repeat for other platforms
     ]
   }
   ```

   **Field reference:**
   - `kind`: Either `"file"` (direct binary) or `"archive"` (tar.gz/zip)
   - `url`: Direct download URL
   - `file`: (archives only) Path to binary within extracted archive
   - `sha256`: SHA256 checksum of the downloaded file
   - `os`: Either `"linux"`, `"macos"`, or `"windows"`
   - `cpu`: Either `"x86_64"` or `"arm64"`

4. **Rebuild bazel_env**

   ```bash
   bazel run //tools:bazel_env
   ```

5. **Update shell environment**

   ```bash
   direnv allow
   # Or manually:
   hash -r
   ```

6. **Test tool availability**

   ```bash
   which kubectl
   kubectl version --client
   ```

#### Platform Support Notes

- **Linux and macOS**: Include both x86_64 and arm64 for cross-platform development
- **macOS-only tools**: If a tool isn't available as a standalone binary for macOS (e.g., Docker CLI), document using Homebrew instead:
  ```bash
  # For macOS developers
  brew install docker
  ```
- **Windows**: Currently not supported in this repository's dev containers

#### Troubleshooting

- **Binary not found after adding**: Ensure the `file` path in archives is correct relative to the extracted archive root
- **Checksum mismatch**: Re-download and recompute the SHA256, ensuring you're hashing the correct file
- **Tool not on PATH**: Run `bazel run //tools:bazel_env` and `direnv allow` again

## Troubleshooting

### direnv Not Working

```bash
# Verify direnv is installed
direnv --version

# Verify hook is set up
echo $DIRENV_DIR

# Re-allow
direnv allow
```

### Tools Not Found

```bash
# Rebuild bazel_env
bazel run //tools:bazel_env

# Re-allow direnv
direnv allow

# Verify tools
which copier
which yq
```

### Build Failures

```bash
# Clean and rebuild
bazel clean
bazel build //...

# Update dependencies
./tools/repin
pnpm install
go mod tidy
```

### Test Failures

```bash
# Run with verbose output
bazel test //path/to:test --test_output=all

# Check test logs
cat bazel-testlogs/path/to/test/test.log
```

## Performance Tips

### Faster Builds

```bash
# Use Aspect CLI
aspect build //...

# Increase parallelism
bazel build //... --jobs=8

# Use disk cache
bazel build //... --disk_cache=~/.cache/bazel
```

### Editor Performance

For large repositories:

1. Exclude bazel-* directories from indexing
2. Use language servers (LSP) instead of full IDE features
3. Disable unused plugins

## Getting Help

If you're stuck:

1. Check [Troubleshooting Guide](../user/troubleshooting.md)
2. Search [GitHub Issues](https://github.com/BlueCentre/blueprint/issues)
3. Ask in [Discussions](https://github.com/BlueCentre/blueprint/discussions)
4. Join [Bazel Slack](https://slack.bazel.build/)

## Resources

- [Bazel Documentation](https://bazel.build/docs)
- [Aspect Workflows](https://docs.aspect.build/)
- [Contributing Guide](contributing.md)
- [Architecture Overview](architecture.md)

## Next Steps

- Read [Contributing Guide](contributing.md)
- Review [Code Style Guide](code-style.md)
- Explore [Testing Guide](testing.md)
- Check [Architecture Documentation](architecture.md)
