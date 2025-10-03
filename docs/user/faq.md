# Frequently Asked Questions (FAQ)

## General Questions

### What is Blueprint?

Blueprint is a polyglot Bazel starter repository that provides a production-ready foundation for building multi-language projects. It includes optimized configurations, developer tools, and best practices for working with Bazel and Aspect Workflows.

### What languages does Blueprint support?

Blueprint supports:
- **Python** (with pip/uv)
- **Go**
- **JavaScript/TypeScript** (with pnpm)
- **Rust** (with Cargo)
- **Java** (with Maven)
- **Kotlin** (with Maven)
- **C/C++**
- **Shell scripts**

### Why Bazel?

Bazel provides:
- **Fast, incremental builds** - Only rebuilds what changed
- **Reproducible builds** - Same inputs = same outputs
- **Multi-language support** - One build system for all languages
- **Scalability** - Works from small projects to large monorepos
- **Remote caching** - Share build artifacts across team
- **Remote execution** - Distribute builds across machines

### What is Aspect Workflows?

Aspect Workflows is a commercial product that enhances the Bazel developer experience with:
- Better CLI commands (`aspect build`, `aspect lint`)
- Improved error messages
- Code generation tools
- Integration with IDEs and CI/CD

Learn more at [aspect.build](https://aspect.build).

## Setup Questions

### Do I need to install all language toolchains?

No, you only need to install toolchains for languages you plan to use. Bazel can download and manage some toolchains automatically (Python, Go, Node.js, Rust).

### Can I use Blueprint without direnv?

Yes, but you'll need to manually add tools to your PATH:

```bash
export PATH="$PWD/bazel-out/bazel_env-opt/bin/tools/bazel_env/bin:$PATH"
```

Or run tools via Bazel:
```bash
bazel run //tools:tool_name
```

### How do I update Blueprint to the latest version?

Blueprint is a template, not a framework. To get updates:

1. Add Blueprint as a remote:
   ```bash
   git remote add blueprint https://github.com/BlueCentre/blueprint.git
   git fetch blueprint
   ```

2. Merge or cherry-pick changes:
   ```bash
   git merge blueprint/main
   # or
   git cherry-pick <commit>
   ```

### Can I remove languages I don't need?

Yes! To remove a language:

1. Remove from `.aspect/cli/config.yaml`
2. Remove dependencies from `MODULE.bazel`
3. Remove language-specific config files
4. Update `.bazelrc` if needed

Example - removing Python:
```bash
# Remove from config.yaml
# Remove bazel_dep(name = "rules_python", ...)
# Delete pyproject.toml, requirements/
```

## Build Questions

### Why is my first build so slow?

The first build downloads dependencies and compiles everything. Subsequent builds are much faster due to caching. You can:
- Use `--disk_cache` for persistent cache
- Configure remote cache to share with team
- Use `aspect build` for better performance

### How do I speed up builds?

1. **Use remote cache** (if available)
2. **Use Aspect CLI** (`aspect build` vs `bazel build`)
3. **Increase parallelism** (`--jobs=N`)
4. **Enable disk cache** (`--disk_cache=~/.bazel/cache`)
5. **Build only what you need** (specific targets, not `//...`)

### What's the difference between `bazel build` and `aspect build`?

`aspect build` is the Aspect CLI's enhanced version that provides:
- Better progress indicators
- Faster performance
- Better error messages
- Interactive mode

Both work with Blueprint, but `aspect build` provides a better experience.

### Why does Bazel download so many dependencies?

Bazel downloads:
- Language toolchains (compilers, runtimes)
- Build rules (rules_python, rules_go, etc.)
- Third-party packages (pip, npm, cargo, maven)
- Build tools (gazelle, buildifier, linters)

These are cached and reused across builds.

## Development Questions

### How do I add a new package/library?

1. Create a directory with source files
2. Run Gazelle to generate BUILD file:
   ```bash
   bazel run //:gazelle
   ```
3. Or create BUILD file manually

### How do I run my application?

```bash
# Run with Bazel
bazel run //path/to:binary

# Or build and run directly
bazel build //path/to:binary
./bazel-bin/path/to/binary
```

### How do I debug build issues?

```bash
# Show commands being run
bazel build //path/to:target -s

# Show why target rebuilt
bazel build //path/to:target --explain=explain.txt

# Show verbose failures
bazel build //path/to:target --verbose_failures

# Clean and rebuild
bazel clean
bazel build //path/to:target
```

### Can I use my language's package manager directly?

Yes! You can use:
- `pip install` (but use `./tools/repin` after)
- `pnpm add` (Bazel uses the lockfile)
- `go get` (then `go mod tidy` and `bazel mod tidy`)
- `cargo add` (Bazel uses Cargo.lock)
- Maven dependencies in MODULE.bazel

### How do I format code?

```bash
# Format all files
format

# Format specific files
format path/to/file.py

# Check formatting
bazel test //tools/format:format_check

# Install pre-commit hook
pre-commit install
```

## Testing Questions

### How do I run a single test?

```bash
bazel test //path/to:test_name
```

### How do I see test output?

```bash
# Show all output
bazel test //path/to:test --test_output=all

# Show only errors
bazel test //path/to:test --test_output=errors

# View log file
cat bazel-testlogs/path/to/test/test.log
```

### Why do tests pass locally but fail in CI?

Common causes:
- **File paths** - CI may have different paths
- **Environment variables** - CI has different env
- **Timing** - Tests may be timing-dependent
- **Resources** - CI may have less memory/CPU
- **Dependencies** - Version differences

Use `--test_output=all` to debug in CI.

### How do I write Bazel tests?

Use language-specific test rules:

```starlark
# Python
py_test(
    name = "my_test",
    srcs = ["my_test.py"],
    deps = [":mylib"],
)

# Go
go_test(
    name = "my_test",
    srcs = ["my_test.go"],
    embed = [":mylib"],
)

# TypeScript
ts_test(
    name = "my_test",
    srcs = ["my.test.ts"],
    deps = [":mylib"],
)
```

## Dependency Questions

### How do I add a Python package?

1. Add to `pyproject.toml`:
   ```toml
   dependencies = ["requests>=2.28.0"]
   ```

2. Update lock files:
   ```bash
   ./tools/repin
   ```

3. Update BUILD files:
   ```bash
   bazel run //:gazelle
   ```

### How do I add an npm package?

```bash
pnpm add package-name
```

BUILD files update automatically via Gazelle.

### How do I add a Go module?

```bash
go get github.com/example/package
go mod tidy
bazel mod tidy
bazel run //:gazelle
```

### Why are my dependencies not updating?

Check that you've:
1. Updated the dependency manifest (pyproject.toml, package.json, etc.)
2. Run the lock/pin command (`./tools/repin`, `pnpm install`, etc.)
3. Updated BUILD files (`bazel run //:gazelle`)
4. Cleaned cache if needed (`bazel clean`)

## CI/CD Questions

### How do I set up CI for Blueprint?

See the [CI/CD Configuration Guide](../admin/ci-cd.md).

Basic GitHub Actions:
```yaml
- uses: actions/checkout@v4
- uses: bazel-contrib/setup-bazel@0.9.0
- run: bazel test //...
```

### Can I use Blueprint with Jenkins/GitLab/CircleCI?

Yes! Blueprint works with any CI system that can run shell commands. Just:
1. Install Bazel (or use Bazelisk)
2. Run `bazel test //...`

### How do I cache Bazel artifacts in CI?

Most CI systems support caching. Cache these directories:
- `~/.cache/bazel` (repository cache)
- Configure remote cache for best performance

## Advanced Questions

### Can I use custom Bazel rules?

Yes! Add rules to your repository or depend on external rule sets via MODULE.bazel.

### How do I cross-compile?

Use `--platforms` flag:

```bash
# Build for Linux ARM64
bazel build --platforms=//tools/platforms:linux_arm64 //path/to:target

# Build for Windows AMD64
bazel build --platforms=//tools/platforms:windows_amd64 //path/to:target
```

### Can I use Blueprint in a monorepo?

Yes! Blueprint is designed for monorepos. You can:
- Have multiple packages
- Mix languages
- Share dependencies
- Use consistent tooling

### How do I migrate an existing project to Blueprint?

1. Start with Blueprint template
2. Copy your source code into packages
3. Run Gazelle to generate BUILD files
4. Migrate dependencies to Bazel-managed
5. Test incrementally

See [Migration Guide](../contributor/migration.md) (coming soon).

### What's the difference between WORKSPACE and MODULE.bazel?

- **WORKSPACE** - Legacy Bazel dependency system
- **MODULE.bazel** - Modern Bazel module system (bzlmod)

Blueprint uses MODULE.bazel (the modern approach).

## Where to Get Help

- **Documentation:** [Blueprint Docs](../README.md)
- **Bazel Docs:** https://bazel.build/docs
- **Aspect Docs:** https://docs.aspect.build/
- **GitHub Issues:** https://github.com/BlueCentre/blueprint/issues
- **Bazel Slack:** https://slack.bazel.build/
- **Stack Overflow:** Tag questions with `bazel`

## Common Error Messages

### "no such package"

Run `bazel run //:gazelle` to generate BUILD files.

### "no such target"

Check that the target exists in the BUILD file. Target names are case-sensitive.

### "Cannot find module"

For Python/npm/Go: Check dependencies are installed and BUILD files are updated.

### "Permission denied"

Check file permissions. For scripts: `chmod +x script.sh`

### "command not found"

Tool not on PATH. Run `bazel run //tools:bazel_env` and `direnv allow`.

### "Lock file out of date"

Update lock files:
- Python: `./tools/repin`
- npm: `pnpm install`
- Go: `go mod tidy && bazel mod tidy`
