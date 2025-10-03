# Release Process

This guide outlines the process for creating and publishing releases of Blueprint.

## Overview

Blueprint follows semantic versioning and maintains a changelog. Releases are automated through GitHub Actions.

## Semantic Versioning

Format: `MAJOR.MINOR.PATCH`

- **MAJOR**: Breaking changes
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes (backward compatible)

Examples:
- `1.0.0` → `1.0.1` - Bug fix
- `1.0.0` → `1.1.0` - New feature
- `1.0.0` → `2.0.0` - Breaking change

## Release Checklist

### Pre-Release

- [ ] All tests passing on main branch
- [ ] Documentation updated
- [ ] CHANGELOG.md updated
- [ ] Version numbers updated (if needed)
- [ ] No known critical bugs

### Creating a Release

1. **Update CHANGELOG.md**

```markdown
## [1.2.0] - 2024-01-15

### Added
- Support for Python 3.12
- New linting rules for Go
- Enhanced documentation

### Changed
- Updated Bazel to 7.0.0
- Improved build performance

### Fixed
- Fixed issue with npm dependencies
- Resolved flaky test in CI

### Breaking Changes
- None
```

2. **Create and Push Tag**

```bash
# Fetch latest
git checkout main
git pull origin main

# Create annotated tag
git tag -a v1.2.0 -m "Release v1.2.0"

# Push tag
git push origin v1.2.0
```

3. **GitHub Release**

GitHub Actions automatically creates a release when a tag is pushed.

Or manually create on GitHub:
- Go to Releases → Draft a new release
- Choose tag: `v1.2.0`
- Release title: `v1.2.0`
- Description: Copy from CHANGELOG.md
- Publish release

### Post-Release

- [ ] Verify release on GitHub
- [ ] Announce release (if needed)
- [ ] Update documentation site
- [ ] Close milestone (if using)

## Release Types

### Patch Release (1.0.0 → 1.0.1)

For bug fixes:

```bash
# Fix the bug
git checkout -b fix/bug-description
# Make changes, commit, and PR

# After merge
git checkout main
git pull
git tag -a v1.0.1 -m "Release v1.0.1 - Bug fixes"
git push origin v1.0.1
```

### Minor Release (1.0.0 → 1.1.0)

For new features:

```bash
# Develop feature
git checkout -b feature/new-feature
# Make changes, commit, and PR

# After merge
git checkout main
git pull
git tag -a v1.1.0 -m "Release v1.1.0 - New features"
git push origin v1.1.0
```

### Major Release (1.0.0 → 2.0.0)

For breaking changes:

1. **Document breaking changes** in CHANGELOG.md
2. **Update migration guide**
3. **Notify users** in advance
4. **Create release**:

```bash
git checkout main
git pull
git tag -a v2.0.0 -m "Release v2.0.0 - Major update"
git push origin v2.0.0
```

## Hotfix Process

For critical bugs in production:

1. **Create hotfix branch from tag:**

```bash
git checkout -b hotfix/1.2.1 v1.2.0
```

2. **Fix the bug:**

```bash
# Make minimal changes
git commit -m "fix: critical bug description"
```

3. **Test thoroughly:**

```bash
bazel test //...
```

4. **Create hotfix release:**

```bash
git tag -a v1.2.1 -m "Hotfix v1.2.1"
git push origin hotfix/1.2.1
git push origin v1.2.1
```

5. **Merge back to main:**

```bash
git checkout main
git merge hotfix/1.2.1
git push origin main
```

## Release Automation

### GitHub Actions Workflow

`.github/workflows/release.yml`:

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Bazel
      uses: bazel-contrib/setup-bazel@0.9.0
    
    - name: Build Release Artifacts
      run: bazel build --config=release //...
    
    - name: Create Release
      uses: softprops/action-gh-release@v1
      with:
        files: |
          bazel-bin/artifacts/*
        generate_release_notes: true
```

## Version Management

### Stamping Builds

Use stamping for version information:

```starlark
# BUILD
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

### Workspace Status Script

`tools/workspace_status.sh`:

```bash
#!/bin/bash
set -euo pipefail

# Get version from git tag
VERSION=$(git describe --tags --always --dirty)

# Get commit hash
COMMIT=$(git rev-parse HEAD)

echo "STABLE_MONOREPO_VERSION ${VERSION}"
echo "STABLE_GIT_COMMIT ${COMMIT}"
```

## CHANGELOG.md Format

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- New features in development

## [1.2.0] - 2024-01-15

### Added
- Feature 1
- Feature 2

### Changed
- Update 1
- Update 2

### Deprecated
- Feature to be removed

### Removed
- Removed feature

### Fixed
- Bug fix 1
- Bug fix 2

### Security
- Security update

## [1.1.0] - 2023-12-01

...
```

## Communication

### Release Notes

Include in GitHub release:

```markdown
## What's New in v1.2.0

### Features
- **Python 3.12 Support**: Full support for latest Python version
- **Enhanced Linting**: New rules for better code quality
- **Improved Docs**: Comprehensive documentation updates

### Improvements
- 20% faster build times
- Better error messages
- Updated dependencies

### Bug Fixes
- Fixed npm dependency resolution
- Resolved CI flakiness

### Breaking Changes
None

### Upgrade Guide
Simply update to the latest version. No migration needed.

### Contributors
Thanks to @user1, @user2, and @user3 for their contributions!
```

### Announcing Releases

Channels to announce:
- GitHub Discussions
- Twitter/Social media
- Mailing list
- Slack/Discord

## Rollback Procedure

If a release has critical issues:

1. **Create new patch release** with fix (preferred)
2. **Or delete tag and release:**

```bash
# Delete remote tag
git push --delete origin v1.2.0

# Delete local tag
git tag -d v1.2.0

# Delete GitHub release (via web interface)
```

3. **Notify users** of the rollback

## Best Practices

1. **Test thoroughly** before releasing
2. **Update docs** with new features
3. **Maintain changelog** consistently
4. **Use semantic versioning** strictly
5. **Automate** release process
6. **Communicate** breaking changes
7. **Provide** upgrade guides
8. **Review** release notes

## Emergency Release

For security vulnerabilities:

1. **Fix privately** (don't disclose in public PR)
2. **Test thoroughly**
3. **Create release** immediately
4. **Notify users** through security advisory
5. **Disclose** vulnerability after patch released

## Resources

- [Semantic Versioning](https://semver.org/)
- [Keep a Changelog](https://keepachangelog.com/)
- [GitHub Releases](https://docs.github.com/en/repositories/releasing-projects-on-github)

## Next Steps

- Review [Maintenance Guide](maintenance.md)
- Check [CI/CD Configuration](ci-cd.md)
- Read [Security Guide](security.md)
