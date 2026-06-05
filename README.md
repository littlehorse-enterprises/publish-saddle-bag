# publish-saddle-bag

A GitHub Actions composite action that builds a Quarkus Slack Saddle Bag with Gradle,
reads OCI annotations from the resulting properties file, and publishes the Docker image
to a container registry.

## Usage

```yaml
- name: Publish Saddle Bag
  uses: littlehorse-enterprises/publish-saddle-bag@v1
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
    images: ghcr.io/${{ github.repository }}/slack-saddle-bag
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `registry` | ✅ | — | Container registry URL (e.g. `ghcr.io`) |
| `username` | ✅ | — | Registry username |
| `password` | ✅ | — | Registry password or token |
| `images` | ✅ | — | Image name(s) passed to `docker/metadata-action` (one per line) |
| `context` | ❌ | `.` | Docker build context path |
| `file` | ❌ | `{context}/Dockerfile` | Path to the Dockerfile |
| `tags` | ❌ | `''` | Tag patterns passed to `docker/metadata-action` (one per line) |
| `labels` | ❌ | `''` | Extra labels passed to `docker/metadata-action` (`KEY=VALUE`, one per line) |
| `annotations` | ❌ | `''` | Extra annotations passed to `docker/metadata-action` (`[TYPE:]KEY=VALUE`, one per line) |
| `docker-build-args` | ❌ | `''` | Docker build arguments (`NAME=VALUE`, one per line) |
| `quarkus-build-args` | ❌ | `''` | Quarkus build arguments (space-separated, e.g. `-Dkey=value`) |

## Outputs

| Output | Description |
|--------|-------------|
| `tags` | Docker image tags produced by the build |
| `digest` | Docker image digest |
| `metadata` | Build result metadata (JSON) |

## Extended example

```yaml
- name: Publish Saddle Bag
  uses: littlehorse-enterprises/publish-saddle-bag@v1
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
    images: ghcr.io/${{ github.repository }}/slack-saddle-bag
    context: bags/slack
    file: bags/slack/src/main/docker/Dockerfile.jvm
    tags: |
      type=semver,pattern={{version}}
      type=ref,event=branch
    labels: |
      org.opencontainers.image.vendor=LittleHorse Enterprises
    annotations: |
      com.example.custom=value
    docker-build-args: |
      APP_VERSION=${{ github.ref_name }}
    quarkus-build-args: -Dquarkus.profile=prod
```
