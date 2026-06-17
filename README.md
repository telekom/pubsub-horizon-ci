# pubsub-horizon-ci

Reusable GitHub Actions workflows for building, testing, and publishing [Horizon](https://github.com/telekom/pubsub-horizon) platform components.

## Workflows

| Workflow | Purpose | Caller trigger |
|----------|---------|----------------|
| `gradle-build.yml` | Build + test Java/Gradle projects with JaCoCo coverage | `push`, `pull_request` |
| `go-test.yml` | Build + test Go projects with DinD services | `push`, `pull_request` |
| `golangci-lint.yml` | Go static analysis | `push`, `pull_request` |
| `docker-publish.yml` | Build Docker image + push to Artifactory | `push tags` |
| `reuse-compliance.yml` | REUSE/SPDX license header check | `push`, `pull_request` |

## Usage

### Java component (e.g., Starlight)

```yaml
name: CI
on:
  push:
    branches: [main]
    tags: ["*"]
  pull_request:
    branches: [main]

jobs:
  build:
    uses: telekom/pubsub-horizon-ci/.github/workflows/gradle-build.yml@main
    with:
      java-version: "21"
      coverage-overall-min: "60"
      coverage-changed-min: "80"

  reuse:
    uses: telekom/pubsub-horizon-ci/.github/workflows/reuse-compliance.yml@main

  publish:
    if: startsWith(github.ref, 'refs/tags/')
    needs: [build]
    uses: telekom/pubsub-horizon-ci/.github/workflows/docker-publish.yml@main
    with:
      component: starlight
      language: java
    secrets:
      REGISTRY_USERNAME: ${{ secrets.ARTIFACTORY_TARDIS_PUSH_USER }}
      REGISTRY_PASSWORD: ${{ secrets.ARTIFACTORY_TARDIS_PUSH_TOKEN }}
```

### Go component (e.g., Golaris)

```yaml
name: CI
on:
  push:
    branches: [main]
    tags: ["*"]
  pull_request:
    branches: [main]

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
    if: startsWith(github.ref, 'refs/tags/')
    needs: [build, lint]
    uses: telekom/pubsub-horizon-ci/.github/workflows/docker-publish.yml@main
    with:
      component: golaris
      language: go
    secrets:
      REGISTRY_USERNAME: ${{ secrets.ARTIFACTORY_TARDIS_PUSH_USER }}
      REGISTRY_PASSWORD: ${{ secrets.ARTIFACTORY_TARDIS_PUSH_TOKEN }}
```

## Registry

Images are pushed to JFrog Artifactory:

```
artifactory.devops.telekom.de/tardis-oci-local/components/horizon/<component>:<tag>
```

Pull without authentication (trusted access from intranet):

```
trusted.artifactory.devops.telekom.de/tardis-oci-local/components/horizon/<component>:<tag>
```

## Secrets required

Each component repo needs these GitHub secrets:

| Secret | Value |
|--------|-------|
| `ARTIFACTORY_TARDIS_PUSH_USER` | Artifactory service account username |
| `ARTIFACTORY_TARDIS_PUSH_TOKEN` | Artifactory service account token |

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
trusted.artifactory.devops.telekom.de/tardis-oci-local/infra/build/pandora-java-21:1.0.0
```

## License

[Apache-2.0](LICENSE)
