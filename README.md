# pubsub-horizon-ci

Reusable GitHub Actions workflows for building, testing, and publishing [Horizon](https://github.com/telekom/pubsub-horizon) platform components.

## Workflows

| Workflow | Purpose |
|----------|---------|
| `gradle-build.yml` | Build + test Java/Gradle projects with JaCoCo coverage |
| `go-test.yml` | Build + test Go projects with DinD services |
| `golangci-lint.yml` | Go static analysis (shared `.golangci.yml` auto-fetched if not present locally) |
| `docker-publish.yml` | Build Docker image + push to Artifactory |
| `reuse-compliance.yml` | REUSE/SPDX license header check |

## Usage

### Java component (e.g., Starlight)

```yaml
name: CI
on:
  push:

jobs:
  build:
    permissions:
      contents: read
      pull-requests: write
    uses: telekom/pubsub-horizon-ci/.github/workflows/gradle-build.yml@main
    with:
      java-version: "21"

  reuse:
    uses: telekom/pubsub-horizon-ci/.github/workflows/reuse-compliance.yml@main

  publish:
    needs: [build]
    uses: telekom/pubsub-horizon-ci/.github/workflows/docker-publish.yml@main
    with:
      component: starlight
      language: java
      build-args: "DOCKER_BASE_IMAGE=artifactory.devops.telekom.de/tardis-oci-local/tardis/infra/build/pandora-java-21:1.0.0"
    secrets:
      REGISTRY_USERNAME: ${{ secrets.ARTIFACTORY_O28M_PUSH_USER }}
      REGISTRY_PASSWORD: ${{ secrets.ARTIFACTORY_O28M_PUSH_TOKEN }}
```

### Go component (e.g., Golaris)

```yaml
name: CI
on:
  push:

jobs:
  build:
    uses: telekom/pubsub-horizon-ci/.github/workflows/go-test.yml@main
    with:
      test-tags: "testing"
      test-flags: "-v -p 1"

  lint:
    uses: telekom/pubsub-horizon-ci/.github/workflows/golangci-lint.yml@main

  reuse:
    uses: telekom/pubsub-horizon-ci/.github/workflows/reuse-compliance.yml@main

  publish:
    needs: [build]
    uses: telekom/pubsub-horizon-ci/.github/workflows/docker-publish.yml@main
    with:
      component: golaris
      language: go
    secrets:
      REGISTRY_USERNAME: ${{ secrets.ARTIFACTORY_O28M_PUSH_USER }}
      REGISTRY_PASSWORD: ${{ secrets.ARTIFACTORY_O28M_PUSH_TOKEN }}
```

## Registry

Images are pushed to JFrog Artifactory:

```
artifactory.devops.telekom.de/tardis-oci-local/tardis/components/horizon/<component>:<tag>
```

### Image tagging

| Git ref | Image tag |
|---------|-----------|
| Tag `3.1.0` | `3.1.0` |
| Branch `main` | `latest` |
| Branch `feat/xyz` | `feat-xyz` |

Tag format matches GitLab's `CI_COMMIT_REF_SLUG` (lowercase, `/` and `.` → `-`, max 63 chars).

### Pulling images

From company intranet (no auth needed):
```
trusted.artifactory.devops.telekom.de/mcicd-internal-oci/tardis/components/horizon/<component>:<tag>
```

From external / GitHub runners (auth required):
```
artifactory.devops.telekom.de/tardis-oci-local/tardis/components/horizon/<component>:<tag>
```

## Secrets

These are org-level GitHub secrets (already configured on all Horizon repos):

| Secret | Value |
|--------|-------|
| `ARTIFACTORY_O28M_PUSH_USER` | Artifactory service account username |
| `ARTIFACTORY_O28M_PUSH_TOKEN` | Artifactory service account token |

## Permissions

Callers that need JaCoCo PR coverage comments must grant:
```yaml
permissions:
  contents: read
  pull-requests: write
```

Reusable workflows cannot escalate permissions — the caller must provide them.

## Components

| Component | Language | Repo |
|-----------|----------|------|
| Starlight | Java | [pubsub-horizon-starlight](https://github.com/telekom/pubsub-horizon-starlight) |
| Comet | Java | [pubsub-horizon-comet](https://github.com/telekom/pubsub-horizon-comet) |
| Galaxy | Java | [pubsub-horizon-galaxy](https://github.com/telekom/pubsub-horizon-galaxy) |
| Pulsar | Java | [pubsub-horizon-pulsar](https://github.com/telekom/pubsub-horizon-pulsar) |
| Golaris | Go | [pubsub-horizon-golaris](https://github.com/telekom/pubsub-horizon-golaris) |
| Vortex | Go | [pubsub-horizon-vortex](https://github.com/telekom/pubsub-horizon-vortex) |
| Quasar | Go | [pubsub-horizon-quasar](https://github.com/telekom/pubsub-horizon-quasar) |

## Base image

Java components use a custom base image with gcompat and DT CA certificates:

```
artifactory.devops.telekom.de/tardis-oci-local/tardis/infra/build/pandora-java-21:1.0.0
```

## License

[Apache-2.0](LICENSE)
