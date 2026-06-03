# Project Overview
`docker_image_pusher` is a GitHub Action-based utility designed to sync Docker images from international registries (like DockerHub, gcr.io, k8s.io, ghcr.io) to Aliyun Container Registry (ACR). This is particularly useful for users in regions with restricted or slow access to international Docker registries.

## Key Technologies
- **GitHub Actions**: Orchestrates the pull, tag, and push process.
- **Docker/Buildx**: Handles image manipulation.
- **Shell Scripting**: Logic for parsing `images.txt`, handling platform-specific tags, and preventing name collisions is implemented directly in the workflow YAML.

## Architecture
The project is minimalist and configuration-driven:
- `.github/workflows/docker-sync.yaml`: Core logic for syncing images from `images.txt`.
- `.github/workflows/docker-build.yaml`: Workflow for building custom Node.js images with Docker CLI.
- `images.txt`: A list of source images to be synced.
- `dockerfiles/`: Contains Dockerfiles for custom image variants (slim, alpine).
- `doc/`: Contains visual guides and specific READMEs for custom images.

---

# Setup and Usage

## Prerequisites
You need an Aliyun account with Container Registry (Personal Edition is sufficient) enabled.

## Configuration (GitHub Secrets)
The following secrets must be configured in your GitHub repository (`Settings -> Secrets and variables -> Actions`):
- `ALIYUN_REGISTRY`: The Aliyun registry endpoint (e.g., `registry.cn-hangzhou.aliyuncs.com`).
- `ALIYUN_NAME_SPACE`: Your Aliyun ACR namespace.
- `ALIYUN_REGISTRY_USER`: Aliyun ACR username.
- `ALIYUN_REGISTRY_PASSWORD`: Aliyun ACR password/access token.

## Adding Images
Modify `images.txt` to include the images you want to sync.
- **Format**: `[--platform=<platform>] <source_image> [<target_name>]`
- **Rules**:
  - **Source Image**: The full image name (e.g., `nginx:latest`, `xhofe/alist:latest`).
  - **Target Name (Optional)**: If provided, this name will be used in Aliyun ACR. If you omit the tag in `target_name`, the source tag will be appended automatically.
  - **Auto-Naming**: If `target_name` is NOT provided:
    - Official images (`library/` or no namespace) keep their original name.
    - Third-party images automatically get their namespace as a prefix (e.g., `xhofe/alist` -> `xhofe_alist`). This prevents naming collisions and improves organization.
- **Examples**:
  - `nginx:latest` -> `nginx:latest`
  - `xhofe/alist:latest` -> `xhofe_alist:latest` (Auto-prefixed)
  - `gcr.io/kaniko-project/executor:v1.14.0` -> `kaniko-project_executor:v1.14.0`
  - `xhofe/alist:latest my-alist` -> `my-alist:latest` (Custom name)
  - `--platform=linux/arm64 alpine:3.18 alpine-arm64` -> `alpine-arm64:3.18`

## Execution
The sync process is triggered by:
1.  **Push**: Any push to the `sync` branch.
2.  **Manual Trigger**: Using the `workflow_dispatch` button in the GitHub Actions tab.
3.  **Schedule (Optional)**: Can be configured in `docker.yaml` via `schedule`.

---

# Development Conventions

## Image Naming Logic
The workflow handles potential conflicts and multi-architecture requirements:
- **Multi-arch**: Supports syncing multiple platforms (e.g., `linux/amd64`, `linux/arm64`) to a single image tag using Docker Manifests. If no platform is specified, all available platforms from the source are synced.
- **Name Collisions**: If multiple images in `images.txt` have the same name but different namespaces (e.g., `xhofe/alist` and `xiaoyaliu/alist`), the original namespace is prepended to the Aliyun image name (e.g., `xhofe_alist`).

## Disk Space Management
The workflow includes a step to maximize build space using `easimon/maximize-build-space` to accommodate large images (up to 40GB). It also proactively cleans up local Docker images after each push to avoid running out of disk space during a batch run.
atch run.
ild space using `easimon/maximize-build-space` to accommodate large images (up to 40GB). It also proactively cleans up local Docker images after each push to avoid running out of disk space during a batch run.
atch run.
