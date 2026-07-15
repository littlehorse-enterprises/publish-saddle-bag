# publish-saddle-bag

A GitHub Actions composite action that builds a Quarkus Saddle Bag with Gradle,
reads OCI annotations from the resulting properties file, publishes the Docker image
to a container registry, and uploads the generated saddle-bag manifests
(`properties`, `yaml`, `json`) as downloadable workflow artifacts.

> **Note:** Only Java-based Quarkus Docker images are supported. Native Quarkus images are not supported yet.

## Usage

```yaml
name: Deploy My Saddle Bag

on:
  push:
    branches:
      - main

permissions:
  contents: read
  packages: write

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v6

      - name: Publish Saddle Bag
        uses: littlehorse-enterprises/publish-saddle-bag@v1
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
          images: ghcr.io/${{ github.repository }}/my-saddle-bag
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `registry` | ✅ | — | Container registry URL (e.g. `ghcr.io`) |
| `username` | ✅ | — | Registry username |
| `password` | ✅ | — | Registry password or token |
| `images` | ✅ | — | Image name(s) passed to `docker/metadata-action` (one per line). The first image is recorded as the saddle bag `docker-image` metadata |
| `version` | ❌ | `${{ github.ref_name }}` | Saddle bag version (overrides `quarkus.application.version`) |
| `prerelease` | ❌ | `false` | Whether this is a prerelease. When `true`, the `latest` tag is not published |
| `working-directory` | ❌ | `.` | Directory where the Gradle build is executed |
| `context` | ❌ | `working-directory` | Docker build context path (defaults to `working-directory`) |
| `dockerfile` | ❌ | `''` | Path to the Dockerfile (defaults to `working-directory/Dockerfile`) |
| `tags` | ❌ | `''` | Extra tag patterns passed to `docker/metadata-action` (one per line). Added on top of the default `latest` and `version` tags |
| `labels` | ❌ | `''` | Extra labels passed to `docker/metadata-action` (`KEY=VALUE`, one per line) |
| `annotations` | ❌ | `''` | Extra annotations passed to `docker/metadata-action` (`[TYPE:]KEY=VALUE`, one per line) |
| `docker-build-args` | ❌ | `''` | Docker build arguments (`NAME=VALUE`, one per line) |
| `quarkus-build-args` | ❌ | `''` | Quarkus build arguments (space-separated, e.g. `-Dkey=value`) |
| `platforms` | ❌ | `linux/amd64,linux/arm64` | Architecture platforms |

## Outputs

| Output | Description |
|--------|-------------|
| `tags` | Docker image tags produced by the build |
| `digest` | Docker image digest |
| `metadata` | Build result metadata (JSON) |

## Artifacts

The action generates the saddle-bag manifest in all three formats and uploads them
as a workflow artifact named `saddle-bag-spec-<project>`, where `<project>` is the
folder name of the `working-directory` input. It is downloadable from the run's summary page:

- `saddle-bag.properties`
- `saddle-bag.yaml`
- `saddle-bag.json`

## Tags

By default the following tags are published:

- `latest` — published unless `prerelease` is `true`
- `version` — the value of the `version` input

Any patterns provided via the `tags` input are added on top of these defaults.

## Extended example

```yaml
- name: Publish Saddle Bag
  uses: littlehorse-enterprises/publish-saddle-bag@v1
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
    images: ghcr.io/${{ github.repository }}/my-saddle-bag
    version: ${{ github.ref_name }}
    prerelease: "false"
    context: bags/my-saddle-bag
    dockerfile: bags/my-saddle-bag/src/main/docker/Dockerfile.jvm
    tags: |
      type=semver,pattern={{version}}
      type=ref,event=branch
    labels: |
      org.opencontainers.image.vendor=My Vendor
    annotations: |
      com.example.custom=value
    docker-build-args: |
      APP_VERSION=${{ github.ref_name }}
    quarkus-build-args: -Dquarkus.profile=prod
    platforms: linux/amd64,linux/arm64
```
