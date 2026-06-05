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
| `build-args` | ❌ | `''` | Docker build arguments (`NAME=VALUE`, one per line) |

## Outputs

| Output | Description |
|--------|-------------|
| `tags` | Docker image tags produced by the build |
| `digest` | Docker image digest |
| `metadata` | Build result metadata (JSON) |

## Steps performed

1. **Checkout** – checks out the calling repository.
2. **Set up JDK 25** – installs Temurin JDK 25 via `actions/setup-java`.
3. **Set up Docker Buildx** – configures an extended Docker builder.
4. **Build Slack Saddle Bag** – runs `gradle build -Dquarkus.littlehorse.saddle.bag.output.format=properties`.
5. **Login to Registry** – authenticates against the target registry.
6. **Read OCI annotations** – reads `bags/slack/build/saddle-bag/saddle-bag.properties` and emits every property prefixed with `io.littlehorse.saddle.bag.` as OCI annotations.
7. **Extract metadata** – runs `docker/metadata-action` to generate tags, labels, and annotations from Git context plus any extra inputs.
8. **Build and push** – builds the Docker image and pushes it with the generated tags, labels, and the merged set of annotations (metadata + OCI properties + user-supplied).

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
      manifest:com.example.custom=value
    build-args: |
      APP_VERSION=${{ github.ref_name }}
```
