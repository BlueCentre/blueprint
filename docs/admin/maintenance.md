# Maintenance Guide

This guide is for administrators and maintainers of Blueprint.

## Table of Contents

- [Regular Maintenance Tasks](#regular-maintenance-tasks)
- [Dependency Updates](#dependency-updates)
- [Managing Tools](#managing-tools)
- [Security Updates](#security-updates)
- [Performance Monitoring](#performance-monitoring)
- [Troubleshooting Common Issues](#troubleshooting-common-issues)

## Regular Maintenance Tasks

### Weekly Tasks

**Review and Merge PRs**
- Review open pull requests
- Run CI checks
- Merge approved PRs
- Close stale PRs

**Monitor Issues**
- Triage new issues
- Label and categorize
- Assign to appropriate maintainers
- Close resolved issues

**Check CI Health**
- Review failed builds
- Investigate flaky tests
- Update CI configurations if needed

### Monthly Tasks

**Update Dependencies**
- Check for Bazel updates
- Update language rules (rules_python, rules_go, etc.)
- Update toolchains
- Run full test suite

**Review Security Alerts**
- Check Dependabot alerts
- Review security advisories
- Update vulnerable dependencies
- Document security fixes

**Performance Review**
- Analyze build times
- Check cache hit rates
- Review resource usage
- Optimize slow targets

### Quarterly Tasks

**Major Updates**
- Plan Bazel version upgrades
- Update documentation
- Review and update examples
- Clean up deprecated features

**Community Engagement**
- Review community feedback
- Prioritize feature requests
- Plan roadmap
- Update CONTRIBUTING.md

## Dependency Updates

### Bazel Version Updates

1. **Check for new Bazel version:**
   ```bash
   # Current version
   cat .bazelversion
   
   # Check for updates
   # https://github.com/bazelbuild/bazel/releases
   ```

2. **Update .bazelversion:**
   ```bash
   echo "7.0.0" > .bazelversion
   ```

3. **Test thoroughly:**
   ```bash
   bazel clean --expunge
   bazel build //...
   bazel test //...
   ```

4. **Update documentation** if needed

5. **Commit and create PR**

### Updating Language Rules

**Python (rules_python):**

```starlark
# MODULE.bazel
bazel_dep(name = "rules_python", version = "1.7.0")  # Updated
```

**Go (rules_go):**

```starlark
# MODULE.bazel
bazel_dep(name = "rules_go", version = "0.58.0")  # Updated
```

**JavaScript (aspect_rules_js):**

```starlark
# MODULE.bazel
bazel_dep(name = "aspect_rules_js", version = "2.7.0")  # Updated
```

**After updating rules:**

```bash
# Clean and rebuild
bazel clean
bazel build //...

# Run all tests
bazel test //...

# Check for deprecation warnings
bazel build //... 2>&1 | grep -i deprecat
```

### Updating Package Dependencies

**Python packages:**

```bash
# Update version in pyproject.toml
vim pyproject.toml

# Repin dependencies
./tools/repin

# Update manifest and BUILD files
bazel run //:gazelle_python_manifest.update
bazel run //:gazelle

# Test
bazel test //...
```

**npm packages:**

```bash
# Update packages
pnpm update

# Or update specific package
pnpm update package-name@latest

# Test
bazel test //...
```

**Go modules:**

```bash
# Update Go dependencies
go get -u ./...
go mod tidy

# Update Bazel
bazel mod tidy
bazel run //:gazelle

# Test
bazel test //...
```

## Managing Tools

### Adding New Tools

1. **Add to tools.lock.json:**

```json
{
  "tools": {
    "newtool": {
      "url": "https://github.com/owner/repo/releases/download/v1.0.0/{file}",
      "files": {
        "linux_amd64": "newtool-linux-amd64.tar.gz",
        "darwin_amd64": "newtool-darwin-amd64.tar.gz",
        "darwin_arm64": "newtool-darwin-arm64.tar.gz"
      }
    }
  }
}
```

2. **Regenerate bazel_env:**

```bash
bazel run //tools:bazel_env
```

3. **Test tool availability:**

```bash
direnv allow
newtool --version
```

### Updating Tools

```bash
# Update version in tools.lock.json
vim tools/tools.lock.json

# Regenerate
bazel run //tools:bazel_env

# Verify
direnv allow
tool --version
```

### Removing Tools

1. Remove from `tools.lock.json`
2. Regenerate: `bazel run //tools:bazel_env`
3. Update documentation

## Security Updates

### Monitoring Security Issues

**GitHub Security Advisories:**
- Review Dependabot alerts
- Check security tab regularly
- Enable notifications for security issues

**Dependency Scanning:**

```bash
# Python - check for vulnerabilities
pip-audit -r requirements/requirements_lock.txt

# npm - check for vulnerabilities
pnpm audit

# Go - check for vulnerabilities
go list -json -m all | nancy sleuth
```

### Applying Security Patches

**For Python dependencies:**

1. Update vulnerable package in `pyproject.toml`
2. Run `./tools/repin`
3. Test: `bazel test //...`
4. Commit with security note

**For npm dependencies:**

1. Run `pnpm update vulnerable-package`
2. Test: `bazel test //...`
3. Commit with security note

**For Go dependencies:**

1. Run `go get -u github.com/org/vulnerable-package`
2. Run `go mod tidy && bazel mod tidy`
3. Test: `bazel test //...`
4. Commit with security note

### Security Best Practices

- **Pin dependencies** - Use lock files
- **Regular updates** - Schedule dependency updates
- **Minimal permissions** - Limit CI/CD permissions
- **Scan regularly** - Use automated security scanners
- **Document vulnerabilities** - Keep security log

## Performance Monitoring

### Build Performance

**Measure build times:**

```bash
# Time full build
time bazel build //...

# Analyze slow actions
bazel analyze-profile profile.json

# Check cache hit rate
bazel info | grep cache
```

**Profile builds:**

```bash
# Generate profile
bazel build //... --profile=profile.json

# Analyze with Bazel's profiler
bazel analyze-profile profile.json --dump=text

# Or open in Chrome
google-chrome chrome://tracing
# Load profile.json
```

### Test Performance

**Identify slow tests:**

```bash
# Run with timing
bazel test //... --test_output=errors --test_summary=detailed

# Find slowest tests
bazel test //... --profile=test-profile.json
bazel analyze-profile test-profile.json
```

**Optimize slow tests:**
- Parallelize test execution
- Use test sharding
- Reduce test data size
- Mock external dependencies

### Cache Performance

**Monitor cache effectiveness:**

```bash
# Check local cache size
du -sh ~/.cache/bazel

# Check remote cache stats (if configured)
# Depends on cache implementation
```

**Optimize caching:**
- Use remote cache for CI/CD
- Configure appropriate cache size
- Clean old cache entries periodically

## Troubleshooting Common Issues

### CI Build Failures

**Check CI logs:**
- Review GitHub Actions output
- Look for error messages
- Check for environment differences

**Common causes:**
- Dependency version mismatches
- Flaky tests
- Resource constraints
- Network issues

**Solutions:**
```bash
# Run CI locally (if possible)
act  # Using nektos/act

# Or replicate CI environment
docker run --rm -it ubuntu:22.04
# Install Bazel and run tests
```

### Flaky Tests

**Identify flaky tests:**

```bash
# Run tests multiple times
bazel test //... --runs_per_test=10

# Check for timing issues
bazel test //path:test --test_timeout=300
```

**Fix flaky tests:**
- Add proper cleanup
- Fix race conditions
- Use deterministic test data
- Mock time-dependent behavior

### Disk Space Issues

**Clean up:**

```bash
# Clean build outputs
bazel clean

# Deep clean (removes all caches)
bazel clean --expunge

# Clean repository cache
rm -rf ~/.cache/bazel/_bazel_$USER

# Check disk usage
bazel info output_base
du -sh $(bazel info output_base)
```

### Version Conflicts

**Python:**
```bash
# Check for conflicts
./tools/repin

# Manually resolve in pyproject.toml if needed
```

**npm:**
```bash
# Check conflicts
pnpm why package-name

# Resolve with overrides in package.json
```

**Go:**
```bash
# Check module graph
go mod graph | grep package-name

# Use replace directive if needed
```

## Backup and Recovery

### Configuration Backup

Important files to backup:
- `.bazelversion`
- `MODULE.bazel`
- `.bazelrc`
- `tools/tools.lock.json`
- Language config files (pyproject.toml, package.json, go.mod)

### Lock File Recovery

If lock files are corrupted:

**Python:**
```bash
./tools/repin
```

**npm:**
```bash
rm pnpm-lock.yaml
pnpm install
```

**Go:**
```bash
rm go.sum
go mod tidy
```

## Health Checks

### Regular Health Check Script

```bash
#!/bin/bash
# health-check.sh

echo "Checking Bazel version..."
bazel version

echo "Checking build..."
bazel build //... || exit 1

echo "Running tests..."
bazel test //... || exit 1

echo "Checking formatting..."
bazel test //tools/format:format_check || exit 1

echo "Running linters..."
aspect lint //... || exit 1

echo "Health check passed!"
```

### Monitoring Metrics

Track these metrics:
- Build success rate
- Average build time
- Test pass rate
- Cache hit rate
- PR merge time
- Issue resolution time

## Documentation Maintenance

### Keep Documentation Updated

**When to update docs:**
- After Bazel version updates
- When adding new features
- When changing workflows
- When fixing bugs

**Documentation checklist:**
- [ ] Update README.md
- [ ] Update relevant guides in docs/
- [ ] Update examples
- [ ] Update diagrams if needed
- [ ] Review for broken links

## Emergency Procedures

### Rolling Back Changes

```bash
# Identify problematic commit
git log --oneline

# Revert commit
git revert <commit-hash>

# Or reset (if not pushed)
git reset --hard HEAD~1

# Push
git push origin main
```

### Critical Bug Fix Process

1. Create hotfix branch
2. Make minimal fix
3. Test thoroughly
4. Fast-track review
5. Merge and deploy
6. Document incident
7. Create follow-up issue for root cause

## Contact Information

**Maintainers:**
- List maintainer names and contacts
- GitHub usernames
- Email addresses (if appropriate)

**Escalation:**
- Who to contact for urgent issues
- Emergency procedures
- On-call rotation (if applicable)

## Next Steps

- Review [CI/CD Configuration](ci-cd.md)
- Check [Release Process](releases.md)
- Read [Security Guide](security.md)
