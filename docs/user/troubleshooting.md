# Troubleshooting Guide

This guide helps you resolve common issues when working with Blueprint.

## Table of Contents

- [Environment Setup Issues](#environment-setup-issues)
- [Build Issues](#build-issues)
- [Dependency Issues](#dependency-issues)
- [Test Issues](#test-issues)
- [Tool Issues](#tool-issues)
- [Performance Issues](#performance-issues)
- [Platform-Specific Issues](#platform-specific-issues)

## Environment Setup Issues

### direnv not loading

**Problem:** `direnv` doesn't automatically set up the environment.

**Solution:**

```bash
# Allow direnv
direnv allow

# If bazel_env bin directory doesn't exist
bazel run //tools:bazel_env
direnv allow
```

### Tools not found on PATH

**Problem:** Commands like `copier`, `yq`, or `pnpm` are not found.

**Solution:**

1. Check if direnv is active:
   ```bash
   echo $DIRENV_DIR
   ```

2. Rebuild bazel_env:
   ```bash
   bazel run //tools:bazel_env
   direnv allow
   ```

3. Verify PATH:
   ```bash
   echo $PATH | tr ':' '\n' | grep bazel-out
   ```

### Wrong Bazel version

**Problem:** Bazel version doesn't match `.bazelversion`.

**Solution:**

If using Bazelisk:
```bash
bazelisk version
```

If using Bazel directly, install Bazelisk:
```bash
# On macOS
brew install bazelisk

# On Linux
wget https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64
chmod +x bazelisk-linux-amd64
sudo mv bazelisk-linux-amd64 /usr/local/bin/bazel
```

## Build Issues

### Build fails with "no such package"

**Problem:** `ERROR: no such package '//path/to/package': BUILD file not found`

**Solution:**

1. Verify BUILD file exists:
   ```bash
   ls path/to/package/BUILD
   ```

2. Generate BUILD files with Gazelle:
   ```bash
   bazel run //:gazelle
   ```

3. Check for `.bazelignore` excluding the path

### Build fails with "no such target"

**Problem:** `ERROR: no such target '//path/to:target'`

**Solution:**

1. Check BUILD file has the target defined
2. Verify target name matches exactly (case-sensitive)
3. Run Gazelle to regenerate targets:
   ```bash
   bazel run //:gazelle
   ```

### Dependency not found

**Problem:** `ERROR: module 'some-module' not found`

**Solution:**

1. Check MODULE.bazel for the dependency
2. Run module update:
   ```bash
   bazel mod deps
   ```

3. For Python/npm/cargo dependencies, see [Dependency Issues](#dependency-issues)

### Compilation errors after update

**Problem:** Build breaks after updating dependencies.

**Solution:**

1. Clean build cache:
   ```bash
   bazel clean --expunge
   ```

2. Update lock files:
   ```bash
   # Python
   ./tools/repin
   
   # JavaScript
   pnpm install
   
   # Go
   go mod tidy
   bazel mod tidy
   ```

3. Rebuild:
   ```bash
   bazel build //...
   ```

### Stale BUILD files

**Problem:** Changes not reflected in build.

**Solution:**

```bash
# Regenerate BUILD files
bazel run //:gazelle

# For Python, update manifest first
bazel run //:gazelle_python_manifest.update
bazel run //:gazelle
```

## Dependency Issues

### Python dependency not found

**Problem:** `ModuleNotFoundError: No module named 'xyz'`

**Solution:**

1. Add to `pyproject.toml`:
   ```toml
   dependencies = [
       "xyz>=1.0.0",
   ]
   ```

2. Update lock files:
   ```bash
   ./tools/repin
   ```

3. Update BUILD files:
   ```bash
   bazel run //:gazelle_python_manifest.update
   bazel run //:gazelle
   ```

### npm package not found

**Problem:** `Cannot find module 'xyz'`

**Solution:**

1. Install package:
   ```bash
   pnpm add xyz
   ```

2. Update BUILD files:
   ```bash
   bazel run //:gazelle
   ```

3. Ensure `node_modules` is linked:
   ```bash
   ls -la node_modules  # Should exist
   ```

### Go module not found

**Problem:** `package github.com/example/pkg is not in std`

**Solution:**

1. Add dependency:
   ```bash
   go get github.com/example/pkg
   ```

2. Update go.mod:
   ```bash
   go mod tidy -v
   ```

3. Update MODULE.bazel:
   ```bash
   bazel mod tidy
   ```

4. Update BUILD files:
   ```bash
   bazel run //:gazelle
   ```

### Dependency version conflicts

**Problem:** Multiple versions of same package requested.

**Solution:**

**For Python:**
```bash
# Check for conflicts
./tools/repin

# Manually resolve in pyproject.toml if needed
```

**For npm:**
```bash
# Check for conflicts
pnpm why package-name

# Update to resolve
pnpm update
```

**For Go:**
```bash
# Check versions
go mod graph | grep package-name

# Use replace directive in go.mod if needed
```

## Test Issues

### Tests fail with "not found"

**Problem:** Test can't find files or dependencies.

**Solution:**

1. Check `data` attribute in BUILD file:
   ```starlark
   py_test(
       name = "test",
       srcs = ["test.py"],
       data = ["testdata/file.txt"],  # Add test data
       deps = [":lib"],
   )
   ```

2. Use runfiles in test code:
   ```python
   from rules_python.python.runfiles import runfiles
   r = runfiles.Create()
   path = r.Rlocation("_main/testdata/file.txt")
   ```

### Tests timeout

**Problem:** Test times out before completing.

**Solution:**

1. Increase timeout in BUILD file:
   ```starlark
   py_test(
       name = "slow_test",
       srcs = ["slow_test.py"],
       timeout = "long",  # short, moderate, long, eternal
   )
   ```

2. Or set via command line:
   ```bash
   bazel test //path/to:test --test_timeout=300
   ```

### Flaky tests

**Problem:** Tests pass/fail inconsistently.

**Solution:**

1. Mark as flaky:
   ```starlark
   py_test(
       name = "flaky_test",
       srcs = ["flaky_test.py"],
       flaky = True,
   )
   ```

2. Run multiple times to debug:
   ```bash
   bazel test //path/to:test --runs_per_test=10
   ```

3. Use `--test_output=all` to see all output:
   ```bash
   bazel test //path/to:test --test_output=all
   ```

## Tool Issues

### Formatter not working

**Problem:** `format` command fails or doesn't format files.

**Solution:**

1. Verify formatter is installed:
   ```bash
   which format
   bazel run //tools:bazel_env
   ```

2. Check file is in supported list:
   ```bash
   # Python: .py files
   # JavaScript: .js, .ts files
   # Go: .go files
   # etc.
   ```

3. Run formatter directly:
   ```bash
   # Python
   ruff format file.py
   
   # JavaScript
   prettier --write file.ts
   ```

### Linter errors

**Problem:** `aspect lint` shows errors.

**Solution:**

1. Try autofix:
   ```bash
   aspect lint --fix //...
   ```

2. Check configuration files:
   - Python: `pyproject.toml` (ruff config)
   - JavaScript: `eslint.config.mjs`
   - Go: Configured in `tools/lint/BUILD`

3. Update baseline (if intentional):
   ```bash
   # For Kotlin
   bazel run //tools/lint:ktlint_baseline.update
   ```

### Gazelle not updating BUILD files

**Problem:** `bazel run //:gazelle` doesn't update files.

**Solution:**

1. Check gazelle directives in BUILD file:
   ```starlark
   # gazelle:prefix github.com/example/project
   ```

2. For Python, update manifest first:
   ```bash
   bazel run //:gazelle_python_manifest.update
   ```

3. Force full update:
   ```bash
   bazel run //:gazelle -- update -r .
   ```

## Performance Issues

### Slow builds

**Problem:** Builds take too long.

**Solution:**

1. Use Aspect CLI for better performance:
   ```bash
   aspect build //...
   ```

2. Enable disk cache:
   ```bash
   # Add to .bazelrc or use flag
   bazel build --disk_cache=~/.bazel/cache //...
   ```

3. Increase parallelism:
   ```bash
   bazel build --jobs=16 //...
   ```

4. Use remote cache (if available):
   ```bash
   bazel build --remote_cache=... //...
   ```

### Disk space issues

**Problem:** Running out of disk space.

**Solution:**

1. Clean build outputs:
   ```bash
   bazel clean
   ```

2. Deep clean:
   ```bash
   bazel clean --expunge
   ```

3. Remove external repositories:
   ```bash
   rm -rf ~/.cache/bazel
   ```

4. Check disk usage:
   ```bash
   bazel info output_base
   du -sh $(bazel info output_base)
   ```

### Memory issues

**Problem:** Build runs out of memory.

**Solution:**

1. Limit memory usage:
   ```bash
   bazel build --local_ram_resources=4096 //...
   ```

2. Reduce parallelism:
   ```bash
   bazel build --jobs=4 //...
   ```

3. Increase system swap space

## Platform-Specific Issues

### macOS: xcrun errors

**Problem:** `xcrun: error: invalid active developer path`

**Solution:**

```bash
# Install Xcode command line tools
xcode-select --install
```

### Linux: C++ compiler not found

**Problem:** `gcc: command not found` or similar.

**Solution:**

```bash
# Ubuntu/Debian
sudo apt-get install build-essential

# Fedora/RHEL
sudo dnf install gcc gcc-c++ make
```

### Windows: Long path issues

**Problem:** Paths exceed Windows limit.

**Solution:**

1. Enable long path support (Windows 10+):
   ```
   Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem
   Set LongPathsEnabled to 1
   ```

2. Use shorter output base:
   ```bash
   bazel --output_base=C:/b build //...
   ```

### Permissions issues

**Problem:** Permission denied errors.

**Solution:**

```bash
# Fix permissions
chmod +x script.sh

# For tools
bazel run //tools:bazel_env
direnv allow
```

## Getting More Help

If your issue isn't covered here:

1. **Search existing issues:**
   - [Blueprint GitHub Issues](https://github.com/BlueCentre/blueprint/issues)
   - [Bazel Issues](https://github.com/bazelbuild/bazel/issues)
   - [Aspect Workflows Issues](https://github.com/aspect-build/aspect-cli/issues)

2. **Check documentation:**
   - [Bazel Documentation](https://bazel.build/docs)
   - [Aspect Workflows Docs](https://docs.aspect.build/)
   - [Rules Documentation](https://bazel.build/rules)

3. **Enable debug output:**
   ```bash
   bazel build //path/to:target -s --verbose_failures
   ```

4. **Ask for help:**
   - [Bazel Slack](https://slack.bazel.build/)
   - [Stack Overflow](https://stackoverflow.com/questions/tagged/bazel)
   - Open a GitHub issue with details

## Diagnostic Commands

Useful commands for debugging:

```bash
# Show Bazel info
bazel info

# Show dependency tree
bazel query --output=graph //path/to:target

# Show action graph
bazel aquery //path/to:target

# Show configuration
bazel config

# Show used .bazelrc flags
bazel --announce_rc build //path/to:target

# Explain build
bazel build //path/to:target --explain=explain.txt
```
