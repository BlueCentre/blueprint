# Security Guide

This guide covers security considerations and best practices for Blueprint projects.

## Table of Contents

- [Security Principles](#security-principles)
- [Dependency Security](#dependency-security)
- [Build Security](#build-security)
- [CI/CD Security](#cicd-security)
- [Container Security](#container-security)
- [Secrets Management](#secrets-management)
- [Security Scanning](#security-scanning)
- [Vulnerability Response](#vulnerability-response)

## Security Principles

### Defense in Depth

Multiple layers of security:

1. **Source code** - Secure coding practices
2. **Dependencies** - Vetted and updated dependencies
3. **Build system** - Sandboxed, reproducible builds
4. **CI/CD** - Secure pipelines
5. **Runtime** - Container security, access controls

### Least Privilege

- Minimal permissions for CI/CD
- Restricted access to secrets
- Limited scope for tokens
- Role-based access control

### Supply Chain Security

- Pin all dependencies with lock files
- Verify dependency checksums
- Use trusted sources only
- Regular security audits

## Dependency Security

### Lock Files

Always use lock files to pin dependencies:

**Python:**
```bash
# Generate lock file
./tools/repin

# Commit lock file
git add requirements/requirements_lock.txt
git commit -m "chore: update Python dependencies"
```

**JavaScript/TypeScript:**
```bash
# Lock file automatically updated
pnpm install

# Commit lock file
git add pnpm-lock.yaml
git commit -m "chore: update npm dependencies"
```

**Go:**
```bash
# Update lock file
go mod tidy

# Commit lock file
git add go.sum
git commit -m "chore: update Go dependencies"
```

### Dependency Scanning

**Python - pip-audit:**
```bash
# Install
pip install pip-audit

# Scan dependencies
pip-audit -r requirements/requirements_lock.txt

# Fix vulnerabilities
pip-audit -r requirements/requirements_lock.txt --fix
```

**JavaScript - pnpm audit:**
```bash
# Scan dependencies
pnpm audit

# Fix vulnerabilities
pnpm audit --fix
```

**Go - govulncheck:**
```bash
# Install
go install golang.org/x/vuln/cmd/govulncheck@latest

# Scan
govulncheck ./...
```

### Dependabot Configuration

`.github/dependabot.yml`:

```yaml
version: 2
updates:
  # Python dependencies
  - package-ecosystem: "pip"
    directory: "/requirements"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
  
  # npm dependencies
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
  
  # Go dependencies
  - package-ecosystem: "gomod"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
  
  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

## Build Security

### Sandboxed Execution

Bazel sandboxes all build actions:

```bash
# Enable stricter sandboxing
bazel build //... --spawn_strategy=sandboxed
```

### Reproducible Builds

Ensure builds are reproducible:

```bash
# Disable stamping for reproducibility
bazel build //... --nostamp

# Verify reproducibility
bazel build //... && mv bazel-bin bazel-bin-1
bazel clean
bazel build //... && mv bazel-bin bazel-bin-2
diff -r bazel-bin-1 bazel-bin-2
```

### Build Isolation

Isolate builds from host system:

```starlark
# .bazelrc
build --incompatible_strict_action_env
build --sandbox_default_allow_network=false
```

## CI/CD Security

### Minimal Permissions

GitHub Actions example:

```yaml
permissions:
  contents: read      # Read repository
  pull-requests: read # Read PRs
  # Don't grant write unless needed
```

For deployments:

```yaml
permissions:
  contents: read
  packages: write     # Publish packages
  id-token: write     # OIDC token
```

### Secrets Management

**Never commit secrets:**

```bash
# Use .gitignore
echo "*.key" >> .gitignore
echo "*.pem" >> .gitignore
echo ".env" >> .gitignore
```

**Use GitHub Secrets:**

```yaml
- name: Use Secret
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: |
    # Use $API_KEY
```

**Use Secret Scanner:**

```bash
# Install git-secrets
git secrets --install

# Scan repository
git secrets --scan
```

### Secure CI Configuration

```yaml
name: CI

on:
  pull_request:
    types: [opened, synchronize, reopened]

# Don't run on forks automatically
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    
    # Minimal permissions
    permissions:
      contents: read
    
    steps:
    - uses: actions/checkout@v4
      with:
        persist-credentials: false  # Don't persist credentials
    
    - name: Build
      run: bazel build //...
      env:
        # No secrets in PRs from forks
        BAZEL_REMOTE_CACHE: ${{ secrets.BAZEL_REMOTE_CACHE || '' }}
```

### Token Scope

Use minimal token scopes:

```yaml
- uses: actions/checkout@v4
  with:
    token: ${{ secrets.GITHUB_TOKEN }}  # Limited scope
    # Don't use personal access token unless necessary
```

## Container Security

### Base Images

Use minimal, trusted base images:

```starlark
# Use distroless for minimal attack surface
oci_image(
    name = "app_image",
    base = "@distroless_python",
    entrypoint = ["/app/main"],
)
```

### Non-Root User

Run containers as non-root:

```dockerfile
# Create non-root user
RUN useradd -m -u 1000 appuser
USER appuser
```

Or in OCI rules:

```starlark
oci_image(
    name = "app_image",
    base = "@distroless_python",
    user = "nonroot",
    entrypoint = ["/app/main"],
)
```

### Image Scanning

Scan images for vulnerabilities:

```bash
# Using Trivy
trivy image myimage:latest

# Using Grype
grype myimage:latest

# Using Snyk
snyk container test myimage:latest
```

### Minimal Layers

Reduce attack surface:

```starlark
# Only include necessary files
oci_image(
    name = "app_image",
    base = "@distroless_python",
    entrypoint = ["/app/main"],
    # Don't include development dependencies
    tars = [":app_only"],
)
```

## Secrets Management

### Environment Variables

```bash
# Don't commit
export API_KEY="secret"

# Use .env files (gitignored)
echo "API_KEY=secret" > .env
```

### Secret Stores

**HashiCorp Vault:**

```yaml
- name: Get Secrets from Vault
  uses: hashicorp/vault-action@v2
  with:
    url: https://vault.example.com
    method: jwt
    secrets: |
      secret/data/api key | API_KEY
```

**AWS Secrets Manager:**

```yaml
- name: Get Secrets
  uses: aws-actions/aws-secretsmanager-get-secrets@v1
  with:
    secret-ids: |
      api-key
    parse-json-secrets: true
```

### Bazel Integration

```starlark
# Don't hardcode secrets in BUILD files
# Use --action_env instead
```

```bash
bazel build //... --action_env=API_KEY="${API_KEY}"
```

## Security Scanning

### Static Analysis

**Python - Bandit:**

```bash
pip install bandit
bandit -r src/
```

**Go - gosec:**

```bash
go install github.com/securego/gosec/v2/cmd/gosec@latest
gosec ./...
```

**JavaScript - eslint-plugin-security:**

```javascript
// eslint.config.mjs
import security from 'eslint-plugin-security';

export default [
    security.configs.recommended,
];
```

### SAST Tools

**Semgrep:**

```bash
# Install
pip install semgrep

# Scan
semgrep --config=auto .
```

**CodeQL:**

```yaml
# .github/workflows/codeql.yml
name: CodeQL

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  analyze:
    runs-on: ubuntu-latest
    
    permissions:
      security-events: write
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v2
      with:
        languages: python, javascript, go
    
    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2
```

### License Compliance

Check dependency licenses:

```bash
# Python
pip-licenses

# npm
pnpm licenses list

# Go
go-licenses check ./...
```

## Vulnerability Response

### Process

1. **Detection**
   - Dependabot alert
   - Security scanner
   - Manual report

2. **Assessment**
   - Verify vulnerability
   - Assess impact
   - Determine severity

3. **Mitigation**
   - Update dependency
   - Apply patch
   - Implement workaround

4. **Testing**
   - Test fix
   - Run security scan
   - Verify no regression

5. **Deployment**
   - Create hotfix release
   - Update documentation
   - Notify users

### Security Advisory

For critical vulnerabilities:

1. **Create security advisory** on GitHub
2. **Fix privately** (don't disclose publicly)
3. **Request CVE** if applicable
4. **Release patch** version
5. **Publish advisory** after fix available
6. **Notify users** through multiple channels

### Disclosure Policy

**Public disclosure timeline:**

- Day 0: Vulnerability reported privately
- Day 0-7: Confirm and assess
- Day 7-30: Develop and test fix
- Day 30: Release patch
- Day 30+: Public disclosure

## Security Checklist

### Code Review

- [ ] No hardcoded secrets
- [ ] Input validation
- [ ] Output encoding
- [ ] Error handling
- [ ] Authentication checks
- [ ] Authorization checks

### Deployment

- [ ] Dependencies scanned
- [ ] Containers scanned
- [ ] Secrets properly managed
- [ ] HTTPS enabled
- [ ] Monitoring configured
- [ ] Incident response plan

### Regular Maintenance

- [ ] Update dependencies weekly
- [ ] Review security advisories
- [ ] Scan containers regularly
- [ ] Audit access logs
- [ ] Test backup/recovery
- [ ] Update security docs

## Best Practices

1. **Never commit secrets** to version control
2. **Pin dependencies** with lock files
3. **Scan regularly** for vulnerabilities
4. **Use minimal base images** for containers
5. **Run as non-root** in containers
6. **Enable sandboxing** in builds
7. **Limit permissions** in CI/CD
8. **Rotate secrets** periodically
9. **Monitor** security alerts
10. **Document** security practices

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [Supply Chain Security](https://slsa.dev/)
- [Container Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

## Reporting Security Issues

To report a security vulnerability:

1. **Do not** open a public issue
2. Email security@example.com (or use GitHub Security Advisory)
3. Include:
   - Description of vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)

We will respond within 48 hours.

## Next Steps

- Review [CI/CD Configuration](ci-cd.md)
- Check [Release Process](releases.md)
- Read [Maintenance Guide](maintenance.md)
