# Reusable Gitea Actions Workflows

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Gitea Actions](https://img.shields.io/badge/Gitea-Actions-orange.svg)](https://docs.gitea.com/usage/actions/overview)
[![actionlint](https://img.shields.io/badge/validated-actionlint-green.svg)](https://github.com/rhysd/actionlint)
[![Renovate](https://img.shields.io/badge/renovate-enabled-blue.svg)](https://renovatebot.com)

Production-ready, reusable Gitea Actions workflows for modern software development. Built with the Single Responsibility Principle, these workflows provide composable building blocks for CI/CD pipelines across your organization.

## Features

- 🔧 **Modular Design** - Each workflow has a single, well-defined responsibility
- 🛡️ **Security First** - Comprehensive security scanning with Trivy, gosec, govulncheck
- 📦 **Production Ready** - Battle-tested with explicit version pinning and error handling
- 🚀 **Easy to Use** - Copy templates and start using in minutes
- 🔄 **Composable** - Mix and match workflows to build custom pipelines
- ✅ **Well Tested** - All workflows validated with actionlint
- 🤖 **Auto Updates** - Renovate bot keeps dependencies up-to-date

## Quick Start

### 1. Simple Go Project

```yaml
# .gitea/workflows/pr.yml
name: Pull Request

on: pull_request

jobs:
  go-ci:
    uses: mandacode-lab/workflows/.gitea/workflows/go-ci.yml@main
    with:
      go-version: "1.25.6"
      coverage-threshold: 80
```

### 2. Go Service with Docker

```yaml
# .gitea/workflows/release.yml
name: Release

on:
  push:
    tags: ['v*']

permissions:
  contents: read
  packages: write

jobs:
  build:
    uses: mandacode-lab/workflows/.gitea/workflows/docker-build.yml@main
    with:
      images: '[{"name": "api", "dockerfile": "Dockerfile"}]'
      registry: gitea.example.com
      repository: ${{ gitea.repository }}
      tag: ${{ gitea.ref_name }}
      tag-prefix: "v"
    secrets:
      registry-username: ${{ gitea.actor }}
      registry-password: ${{ secrets.GITEA_TOKEN }}
```

### 3. Full Stack with Kubernetes

```yaml
jobs:
  docker-build:
    uses: mandacode-lab/workflows/.gitea/workflows/docker-build.yml@main
    # ... docker configuration

  helm-package:
    needs: docker-build
    uses: mandacode-lab/workflows/.gitea/workflows/helm-package.yml@main
    # ... helm configuration
```

## Available Workflows

### Docker Workflows

| Workflow | Description | Use Case |
|----------|-------------|----------|
| [docker-test.yml](.gitea/workflows/docker-test.yml) | Dockerfile linting and security scanning | PR validation |
| [docker-build.yml](.gitea/workflows/docker-build.yml) | Build and push to OCI registries | Gitea Registry, Docker Hub, Harbor |
| [docker-build-ecr.yml](.gitea/workflows/docker-build-ecr.yml) | Build and push to AWS ECR | AWS deployments |

**Key Features:**
- Hadolint v2.14.0 for Dockerfile best practices
- Trivy 0.28.0 for vulnerability scanning
- Multi-platform builds (linux/amd64, linux/arm64)
- Automatic repository creation (ECR)

### Go Workflows

| Workflow | Description |
|----------|-------------|
| [go-ci.yml](.gitea/workflows/go-ci.yml) | Complete Go CI pipeline |

**Includes:**
- Build verification
- Test execution with coverage reporting
- golangci-lint v2.7.2
- Security scanning (gosec 2.22.11, govulncheck)
- Optional Codecov integration

### Helm Workflows

| Workflow | Description |
|----------|-------------|
| [helm-test.yml](.gitea/workflows/helm-test.yml) | Comprehensive Helm chart testing |
| [helm-package.yml](.gitea/workflows/helm-package.yml) | Package and publish to OCI registries |
| [helm-package-ecr.yml](.gitea/workflows/helm-package-ecr.yml) | Package and publish to AWS ECR |

**Features:**
- Helm lint and template validation
- kubeconform v0.7.0 for Kubernetes schema validation
- Kind-based installation testing
- Library chart support
- Optional GPG signing

### Validation

| Workflow | Description |
|----------|-------------|
| [workflow-lint.yml](.gitea/workflows/workflow-lint.yml) | Workflow validation and linting |

**Validates:**
- Gitea Actions syntax (actionlint)
- YAML formatting (yamllint)
- Deprecated syntax detection
- Workflow structure

## Templates

Pre-built templates for common scenarios are available in the [`templates/`](templates/) directory:

- **PR Workflows**
  - `common-go-project-pr.yml` - Go + Docker + Helm
  - `migration-image-pr.yml` - Migration image validation

- **Release Workflows**
  - `common-go-project-release.yml` - Gitea Registry deployment
  - `common-go-project-release-ecr.yml` - AWS ECR deployment
  - `migration-image-release.yml` - Migration image to Gitea Registry
  - `migration-image-release-ecr.yml` - Migration image to AWS ECR

### Using Templates

```bash
# Copy template to your project
cp templates/common-go-project-pr.yml your-project/.gitea/workflows/pr.yml

# Customize for your project
# - Update repository references
# - Adjust versions and paths
# - Configure coverage thresholds
# - Update registry URL to your Gitea instance
```

## Documentation

### Docker Build Configuration

<details>
<summary>Click to expand</summary>

**Multi-image builds:**
```yaml
with:
  images: |
    [
      {"name": "api", "dockerfile": "Dockerfile"},
      {"name": "worker", "dockerfile": "services/worker/Dockerfile"},
      {"name": "migration", "dockerfile": "docker/migration/Dockerfile"}
    ]
```

**Custom build arguments:**
```yaml
with:
  build-args: |
    VERSION=${{ gitea.ref_name }}
    BUILD_DATE=${{ gitea.event.head_commit.timestamp }}
    GIT_COMMIT=${{ gitea.sha }}
```

**Multi-platform builds:**
```yaml
with:
  platforms: "linux/amd64,linux/arm64,linux/arm/v7"
```

</details>

### AWS ECR Authentication

<details>
<summary>Click to expand</summary>

**OIDC (Recommended):**
```yaml
permissions:
  id-token: write
  contents: read

jobs:
  build:
    uses: mandacode-lab/workflows/.gitea/workflows/docker-build-ecr.yml@main
    secrets:
      aws-role-arn: arn:aws:iam::123456789012:role/GiteaActionsRole
```

**Access Keys:**
```yaml
secrets:
  aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
  aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

</details>

### Helm Chart Configuration

<details>
<summary>Click to expand</summary>

**Multiple charts:**
```yaml
with:
  charts: |
    [
      {"dir": "charts/api"},
      {"dir": "charts/worker"},
      {"dir": "charts/frontend"}
    ]
```

**With signing:**
```yaml
with:
  enable-signing: true
secrets:
  gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY }}
  gpg-key-name: "your.email@example.com"
  gpg-passphrase: ${{ secrets.GPG_PASSPHRASE }}
```

</details>

## Architecture

### Design Principles

**Single Responsibility Principle (SRP)**

Each workflow performs one specific task:
- `docker-test.yml` → Test Dockerfiles
- `docker-build.yml` → Build and push images
- `go-ci.yml` → Go continuous integration
- `helm-test.yml` → Test Helm charts
- `helm-package.yml` → Package and publish charts

**Composition Over Inheritance**

Build complex pipelines by composing simple workflows:

```yaml
jobs:
  # Run in parallel
  go-ci:
    uses: mandacode-lab/workflows/.gitea/workflows/go-ci.yml@main

  # Run after go-ci
  docker-build:
    needs: go-ci
    uses: mandacode-lab/workflows/.gitea/workflows/docker-build.yml@main

  # Run after docker-build
  helm-package:
    needs: docker-build
    uses: mandacode-lab/workflows/.gitea/workflows/helm-package.yml@main
```

## Verified Tools

All tools are industry-standard, actively maintained, and widely adopted:

| Tool | Version | Purpose |
|------|---------|---------|
| [Hadolint](https://github.com/hadolint/hadolint) | v2.14.0 | Dockerfile linting |
| [Trivy](https://github.com/aquasecurity/trivy) | 0.28.0 | Security scanning |
| [golangci-lint](https://github.com/golangci/golangci-lint) | v2.7.2 | Go linting |
| [gosec](https://github.com/securego/gosec) | 2.22.11 | Go security |
| [govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck) | latest | Go vulnerabilities |
| [kubeconform](https://github.com/yannh/kubeconform) | 0.7.0 | K8s validation |
| [actionlint](https://github.com/rhysd/actionlint) | latest | Actions linting |
| [yamllint](https://github.com/adrienverge/yamllint) | latest | YAML linting |

## Security Features

- **Vulnerability Scanning**
  - Trivy for container images
  - gosec for Go code
  - govulncheck for Go dependencies

- **Best Practices**
  - Least privilege permissions
  - Secret management
  - Immutable tags for production
  - Multi-platform builds for consistency

## Gitea-Specific Notes

### Environment Variables

Gitea Actions uses `GITEA_*` environment variables instead of `GITHUB_*`:

| GitHub | Gitea |
|--------|-------|
| `$GITHUB_OUTPUT` | `$GITEA_OUTPUT` |
| `$GITHUB_STEP_SUMMARY` | `$GITEA_STEP_SUMMARY` |
| `$GITHUB_PATH` | `$GITEA_PATH` |
| `${{ github.repository }}` | `${{ gitea.repository }}` |
| `${{ github.actor }}` | `${{ gitea.actor }}` |
| `${{ github.ref_name }}` | `${{ gitea.ref_name }}` |

### Token Configuration

Use `GITEA_TOKEN` secret instead of `GITHUB_TOKEN`:

```yaml
secrets:
  registry-username: ${{ gitea.actor }}
  registry-password: ${{ secrets.GITEA_TOKEN }}
```

## Best Practices

### Version Pinning

```yaml
# ❌ Not recommended - uses latest main
uses: mandacode-lab/workflows/.gitea/workflows/go-ci.yml@main

# ✅ Recommended - pins to specific version
uses: mandacode-lab/workflows/.gitea/workflows/go-ci.yml@v1.0.0
```

### Minimal Permissions

```yaml
permissions:
  contents: read        # Read repository contents
  packages: write       # Push to container registry
```

### Conditional Execution

```yaml
jobs:
  docker-test:
    if: contains(gitea.event.pull_request.changed_files, 'Dockerfile')
    uses: mandacode-lab/workflows/.gitea/workflows/docker-test.yml@main
```

### Parallel Execution

```yaml
jobs:
  # These run in parallel
  go-ci:
    uses: mandacode-lab/workflows/.gitea/workflows/go-ci.yml@main

  docker-test:
    uses: mandacode-lab/workflows/.gitea/workflows/docker-test.yml@main

  helm-test:
    uses: mandacode-lab/workflows/.gitea/workflows/helm-test.yml@main
```

## Troubleshooting

### Common Issues

**Workflow not found**
```
Error: Unable to resolve action `mandacode-lab/workflows/.gitea/workflows/go-ci.yml@main`
```
- Ensure the repository is accessible
- Verify the workflow file path is correct
- Check that the branch/tag exists

**Permission denied**
```
Error: Resource not accessible by integration
```
- Check the `permissions` section in your workflow
- Ensure required permissions are granted
- Verify GITEA_TOKEN has necessary scopes

**Secret not found**
```
Error: Secret AWS_ROLE_ARN not found
```
- Add secrets in Repository Settings → Secrets
- Verify secret names match exactly (case-sensitive)

### Getting Help

1. Review workflow source code for input parameters
2. Check the [Gitea Actions documentation](https://docs.gitea.com/usage/actions/overview)
3. Open an issue with workflow logs

## Renovate Bot

This repository uses [Renovate](https://renovatebot.com) to automatically keep dependencies up-to-date.

### What Renovate Updates

| Type | Examples |
|------|----------|
| GitHub Actions | `uses: actions/checkout@v6` |
| Docker Images | Base images in Dockerfiles |
| Go Version | `go-version: "1.25.6"` |
| Helm Version | `helm-version: "v4.0.4"` |
| Kubernetes Version | `kubernetes-version: "v1.34.0"` |
| Lint Tools | golangci-lint, hadolint, kubeconform, gosec |

### Configuration

Renovate is configured in [`renovate.json`](renovate.json):

- **Schedule**: Weekly updates on Monday before 9am (Asia/Seoul)
- **Grouping**: Related updates are grouped together
- **Pin Digests**: Actions are pinned to SHA digests for security
- **Auto-merge**: Patch updates for dev dependencies are auto-merged
- **Approval Required**: Major updates require manual approval

### Manual Trigger

You can trigger Renovate manually by:

1. Creating an issue with the title `renovate` or using the dependency dashboard
2. Using the Renovate bot commands in issues/PRs

### Disable Renovate

To disable Renovate for this repository, add to `renovate.json`:

```json
{
  "enabled": false
}
```

## Contributing

Contributions are welcome! Please follow these guidelines:

### Adding a New Workflow

1. Follow the Single Responsibility Principle
2. Use verified, well-known tools only
3. Pin all versions explicitly
4. Add comprehensive documentation
5. Include usage examples in the header
6. Validate with `actionlint`

### Updating Existing Workflows

1. Maintain backward compatibility when possible
2. Update documentation and examples
3. Test thoroughly before merging
4. Consider creating a new version for breaking changes

### Workflow Template

```yaml
name: Workflow Name

# Brief description
#
# Usage example:
#   jobs:
#     workflow-name:
#       uses: mandacode-lab/workflows/.gitea/workflows/workflow-name.yml@main

on:
  workflow_call:
    inputs:
      # Document all inputs
    secrets:
      # Document all secrets

permissions:
  # Minimal required permissions

jobs:
  # Implementation
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Inspired by reusable workflows patterns
- Built with best practices from the community
- Tools provided by their respective maintainers

---

**Made with ❤️ by mandacode-lab**

For more information, see the [Gitea Actions documentation](https://docs.gitea.com/usage/actions/overview).
