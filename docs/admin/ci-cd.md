# CI/CD Configuration

This guide covers setting up Continuous Integration and Continuous Deployment for Blueprint projects.

## Table of Contents

- [Overview](#overview)
- [GitHub Actions](#github-actions)
- [Other CI Systems](#other-ci-systems)
- [Caching Strategies](#caching-strategies)
- [Remote Execution](#remote-execution)
- [Deployment](#deployment)

## Overview

Blueprint projects use Bazel which provides excellent CI/CD integration:

- **Reproducible builds** - Same results across environments
- **Incremental builds** - Only rebuild what changed
- **Remote caching** - Share artifacts across CI runs
- **Remote execution** - Distribute builds across machines

## GitHub Actions

### Basic Setup

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Bazel
      uses: bazel-contrib/setup-bazel@0.9.0
      with:
        bazelisk-cache: true
        disk-cache: ${{ github.workflow }}
        repository-cache: true
    
    - name: Build
      run: bazel build //...
    
    - name: Test
      run: bazel test //...
    
    - name: Lint
      run: bazel run @aspect_cli//cli -- lint //...
```

### Advanced Configuration

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Bazel
      uses: bazel-contrib/setup-bazel@0.9.0
      with:
        bazelisk-cache: true
        disk-cache: ${{ github.workflow }}
        repository-cache: true
    
    - name: Configure Bazel
      run: |
        echo "build --remote_cache=https://storage.googleapis.com/your-cache" >> .bazelrc.ci
        echo "build --google_default_credentials" >> .bazelrc.ci
    
    - name: Build
      run: bazel --bazelrc=.bazelrc.ci build //...
    
    - name: Test
      run: bazel --bazelrc=.bazelrc.ci test //... --test_output=errors
    
    - name: Lint
      run: bazel run @aspect_cli//cli -- lint //...
    
    - name: Upload Test Results
      if: always()
      uses: actions/upload-artifact@v4
      with:
        name: test-results
        path: bazel-testlogs/
    
    - name: Upload Coverage
      if: github.event_name == 'push'
      uses: codecov/codecov-action@v4
      with:
        files: ./bazel-out/_coverage/_coverage_report.dat
```

### Matrix Builds

Test multiple platforms:

```yaml
jobs:
  build-and-test:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        
    runs-on: ${{ matrix.os }}
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Bazel
      uses: bazel-contrib/setup-bazel@0.9.0
    
    - name: Build and Test
      run: |
        bazel build //...
        bazel test //...
```

### Conditional Jobs

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: bazel-contrib/setup-bazel@0.9.0
    - run: bazel run @aspect_cli//cli -- lint //...
  
  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: bazel-contrib/setup-bazel@0.9.0
    - run: bazel test //...
  
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: bazel-contrib/setup-bazel@0.9.0
    - run: bazel run //deploy:push_images
```

## Other CI Systems

### GitLab CI

`.gitlab-ci.yml`:

```yaml
image: ubuntu:22.04

stages:
  - build
  - test
  - deploy

before_script:
  - apt-get update && apt-get install -y curl
  - curl -L https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64 -o /usr/local/bin/bazel
  - chmod +x /usr/local/bin/bazel

build:
  stage: build
  script:
    - bazel build //...
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - ~/.cache/bazel

test:
  stage: test
  script:
    - bazel test //...
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - ~/.cache/bazel

deploy:
  stage: deploy
  script:
    - bazel run //deploy:push_images
  only:
    - main
```

### Jenkins

`Jenkinsfile`:

```groovy
pipeline {
    agent any
    
    environment {
        BAZEL_VERSION = '7.0.0'
    }
    
    stages {
        stage('Setup') {
            steps {
                sh '''
                    wget https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64
                    chmod +x bazelisk-linux-amd64
                    sudo mv bazelisk-linux-amd64 /usr/local/bin/bazel
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh 'bazel build //...'
            }
        }
        
        stage('Test') {
            steps {
                sh 'bazel test //...'
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'bazel run //deploy:push_images'
            }
        }
    }
    
    post {
        always {
            junit 'bazel-testlogs/**/test.xml'
        }
    }
}
```

### CircleCI

`.circleci/config.yml`:

```yaml
version: 2.1

jobs:
  build-and-test:
    docker:
      - image: cimg/base:stable
    
    steps:
      - checkout
      
      - run:
          name: Install Bazelisk
          command: |
            curl -L https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64 -o ~/bin/bazel
            chmod +x ~/bin/bazel
      
      - restore_cache:
          keys:
            - bazel-cache-{{ .Branch }}-{{ .Revision }}
            - bazel-cache-{{ .Branch }}-
            - bazel-cache-
      
      - run:
          name: Build
          command: bazel build //...
      
      - run:
          name: Test
          command: bazel test //...
      
      - save_cache:
          key: bazel-cache-{{ .Branch }}-{{ .Revision }}
          paths:
            - ~/.cache/bazel

workflows:
  version: 2
  build-and-test:
    jobs:
      - build-and-test
```

## Caching Strategies

### Local Disk Cache

For CI runners with persistent disks:

```yaml
- name: Configure Disk Cache
  run: |
    echo "build --disk_cache=~/.cache/bazel" >> .bazelrc.ci
```

### Remote Cache (Google Cloud Storage)

```yaml
- name: Setup Remote Cache
  env:
    GCS_CACHE: gs://your-bucket/bazel-cache
  run: |
    echo "build --remote_cache=${GCS_CACHE}" >> .bazelrc.ci
    echo "build --google_default_credentials" >> .bazelrc.ci
```

### Remote Cache (AWS S3)

```yaml
- name: Setup Remote Cache
  env:
    AWS_REGION: us-east-1
    S3_BUCKET: your-bucket
  run: |
    echo "build --remote_cache=https://${S3_BUCKET}.s3.${AWS_REGION}.amazonaws.com/bazel-cache" >> .bazelrc.ci
    echo "build --remote_upload_local_results=true" >> .bazelrc.ci
```

### Aspect Build Cloud

For commercial remote cache:

```yaml
- name: Setup Aspect Build Cloud
  env:
    ASPECT_API_KEY: ${{ secrets.ASPECT_API_KEY }}
  run: |
    echo "build --remote_cache=grpcs://remote.aspect.build" >> .bazelrc.ci
    echo "build --remote_header=x-aspect-api-key=${ASPECT_API_KEY}" >> .bazelrc.ci
```

## Remote Execution

### BuildBuddy

```yaml
- name: Configure BuildBuddy
  env:
    BUILDBUDDY_API_KEY: ${{ secrets.BUILDBUDDY_API_KEY }}
  run: |
    echo "build --remote_executor=grpcs://remote.buildbuddy.io" >> .bazelrc.ci
    echo "build --remote_header=x-buildbuddy-api-key=${BUILDBUDDY_API_KEY}" >> .bazelrc.ci
    echo "build --remote_timeout=600" >> .bazelrc.ci
```

### Google Cloud Build

```yaml
- name: Configure GCE Remote Execution
  run: |
    echo "build --remote_executor=remotebuildexecution.googleapis.com" >> .bazelrc.ci
    echo "build --remote_instance_name=projects/your-project/instances/default_instance" >> .bazelrc.ci
    echo "build --google_default_credentials" >> .bazelrc.ci
```

## Deployment

### Container Images

Build and push Docker images:

```yaml
- name: Build and Push Images
  if: github.ref == 'refs/heads/main'
  env:
    DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
    DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
  run: |
    echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin
    bazel run //apps/server:image.push
```

### Kubernetes Deployment

```yaml
- name: Deploy to Kubernetes
  if: github.ref == 'refs/heads/main'
  env:
    KUBECONFIG: ${{ secrets.KUBECONFIG }}
  run: |
    kubectl apply -f k8s/
    kubectl rollout status deployment/app
```

### Cloud Functions

```yaml
- name: Deploy to Cloud Functions
  if: github.ref == 'refs/heads/main'
  env:
    GCP_PROJECT: ${{ secrets.GCP_PROJECT }}
    GCP_SA_KEY: ${{ secrets.GCP_SA_KEY }}
  run: |
    echo "${GCP_SA_KEY}" | base64 -d > /tmp/key.json
    gcloud auth activate-service-account --key-file=/tmp/key.json
    bazel run //functions:deploy
```

## Best Practices

### 1. Use Bazelisk

Always use Bazelisk in CI to ensure correct Bazel version:

```yaml
- name: Install Bazelisk
  run: |
    curl -L https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64 -o /usr/local/bin/bazel
    chmod +x /usr/local/bin/bazel
```

### 2. Enable Remote Cache

Significantly speeds up CI builds:

```yaml
- name: Configure Remote Cache
  run: |
    echo "build --remote_cache=..." >> .bazelrc.ci
```

### 3. Fail Fast

Use `--keep_going=false` to stop on first error:

```yaml
- name: Build
  run: bazel build //... --keep_going=false
```

### 4. Test Output

Only show errors to keep logs clean:

```yaml
- name: Test
  run: bazel test //... --test_output=errors
```

### 5. Parallel Execution

Adjust parallelism based on CI resources:

```yaml
- name: Build
  run: bazel build //... --jobs=4
```

### 6. Timeouts

Set appropriate timeouts:

```yaml
- name: Test
  timeout-minutes: 30
  run: bazel test //...
```

### 7. Artifacts

Upload important artifacts:

```yaml
- name: Upload Test Logs
  if: failure()
  uses: actions/upload-artifact@v4
  with:
    name: test-logs
    path: bazel-testlogs/
```

## Monitoring and Debugging

### Build Event Service

Configure BES for build insights:

```yaml
- name: Configure BES
  run: |
    echo "build --bes_backend=grpcs://bes.example.com" >> .bazelrc.ci
    echo "build --bes_results_url=https://results.example.com" >> .bazelrc.ci
```

### Build Profile

Generate and upload build profiles:

```yaml
- name: Build with Profile
  run: bazel build //... --profile=profile.json

- name: Upload Profile
  uses: actions/upload-artifact@v4
  with:
    name: build-profile
    path: profile.json
```

### Verbose Logging

For debugging CI issues:

```yaml
- name: Debug Build
  run: bazel build //... --verbose_failures --subcommands
```

## Security

### Secrets Management

Never commit secrets:

```yaml
- name: Use Secrets
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: |
    echo "build --action_env=API_KEY=${API_KEY}" >> .bazelrc.ci
```

### Dependency Scanning

```yaml
- name: Security Scan
  run: |
    # Python
    pip-audit -r requirements/requirements_lock.txt
    
    # npm
    pnpm audit
    
    # Go
    go list -json -m all | nancy sleuth
```

## Performance Tips

1. **Use remote cache** - Avoid rebuilding unchanged code
2. **Enable BES** - Monitor and optimize builds
3. **Parallel jobs** - Use `--jobs=N` appropriately
4. **Remote execution** - Distribute work across machines
5. **Clean selectively** - Avoid `bazel clean --expunge`
6. **Cache dependencies** - Cache `~/.cache/bazel`

## Resources

- [Bazel CI Best Practices](https://bazel.build/configure/best-practices)
- [GitHub Actions for Bazel](https://github.com/bazel-contrib/setup-bazel)
- [BuildBuddy Documentation](https://www.buildbuddy.io/docs/introduction/)
- [Aspect Build Documentation](https://docs.aspect.build/)

## Next Steps

- Review [Release Process](releases.md)
- Check [Maintenance Guide](maintenance.md)
- Explore [Security Guide](security.md)
