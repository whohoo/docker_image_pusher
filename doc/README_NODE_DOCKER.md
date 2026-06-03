# Node.js with Docker CLI

A lightweight Docker image based on official Node.js images, equipped with the native Docker CLI. This image is specifically designed for CI/CD runners (like GitHub Actions self-hosted runners or GitLab CI) that need to execute Docker commands while running Node.js tasks.

## Supported Tags

This repository provides multiple variants to suit your needs:

- **Node 24 (Latest Stable)**
  - `latest`, `24-slim`, `latest-slim`: Based on `node:24-slim` (Debian).
  - `24-alpine`, `latest-alpine`: Based on `node:24-alpine` (Alpine Linux).
- **Node 22 (LTS)**
  - `22-slim`: Based on `node:22-slim` (Debian).
  - `22-alpine`: Based on `node:22-alpine` (Alpine Linux).

## Architecture Support

All images are built for multiple architectures:

- `linux/amd64` (Standard servers)
- `linux/arm64` (Apple Silicon, Raspberry Pi, ARM cloud instances)

## Features

- **Pre-installed Docker CLI**: Run `docker ps`, `docker build`, etc., directly from your Node.js environment.
- **Optimized Size**: Uses `slim` and `alpine` variants to minimize footprint.
- **Security**: Regularly updated base images.

## Usage

### In CI/CD

Ideal for steps that require both Node.js environment and Docker interactions (e.g., building and pushing another Docker image after a Node.js build).

```yaml
# Example GitHub Action
jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: whohoo/node-docker:latest
    steps:
      - name: Build App
        run: npm install && npm run build
      - name: Build Docker Image
        run: docker build -t my-app .
```

### Locally

```bash
docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock your-docker-hub-user/node-docker:24-alpine docker version
```

*Note: Mounting `/var/run/docker.sock` is required to interact with the host's Docker daemon.*

## Maintenance

Maintained by: whohoo.ho@gmail.com
Source: [GitHub Repository](https://github.com/whohoo/docker_image_pusher)
