# Project Overview
`docker_image_pusher` is a GitHub Action-based utility designed to sync Docker images from international registries (like DockerHub, gcr.io, k8s.io, ghcr.io) to Aliyun Container Registry (ACR). This is particularly useful for users in regions with restricted or slow access to international Docker registries.

## Key Technologies
- **GitHub Actions**: Orchestrates the pull, tag, and push process.
- **Docker/Buildx**: Handles image manipulation.
- **Shell Scripting**: Logic for parsing `images.txt`, handling platform-specific tags, and preventing name collisions is implemented directly in the workflow YAML.

## Architecture
The project is minimalist and configuration-driven:
- `.github/workflows/docker.yaml`: Contains the core logic for processing images.
- `images.txt`: A list of source images to be synced.
- `doc/`: Contains visual guides for setup.

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
- **Format**: `[--platform=<platform>] <image>[:tag]`
- **Examples**:
  - `nginx:latest`
  - `--platform=linux/arm64 alpine:3.18`
  - `gcr.io/google-containers/pause:3.9`

## Execution
The sync process is triggered by:
1.  **Push**: Any push to the `main` branch.
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
