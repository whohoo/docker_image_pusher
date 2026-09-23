# AGENTS.md

This is a GitHub Actions-driven repo (no app code). All logic lives inside workflow YAML; there is no build, test, or lint tooling to run. The main docs are in Chinese (README.md), so verify behavior against `.github/workflows/` YAML, not prose.

## Branch names are the trigger mechanism

Nothing runs on `main`. Push to specific branches (or use `workflow_dispatch`):

- `sync` branch → `.github/workflows/docker-sync.yaml`: syncs every non-commented line of `images.txt` to Aliyun ACR.
- `build-node-docker` branch → `.github/workflows/docker-build.yaml`: builds/pushes the `node-docker:*-slim/*-alpine` multi-arch images to Docker Hub (tags `latest`, `latest-slim`, `latest-alpine`, `22/24-slim/alpine`).

To test a change to sync logic you cannot run it locally; commit to `sync` and watch the workflow, or trigger via `workflow_dispatch` in the Actions tab.

## images.txt format

One image per line. Grammar: `[--platform=<platform>] <source_image> [<target_name>]`. Lines starting with `#` are comments. `@sha256:` digests are stripped from source images.

Naming rules (implemented in `docker-sync.yaml`, inlined bash):
- No target_name given: official `library/` images keep their name; third-party images get their namespace as a prefix with `/` → `_` (e.g. `xhofe/alist` → `xhofe_alist`). This is the collision-avoidance convention.
- `target_name` given: used verbatim; if it has no `:tag`, the source tag is appended.
- No `--platform` → `docker buildx imagetools create` copies only the manifest (fast, no layer download). With `--platform` → `docker buildx build --platform ... --push`, which fails (and stops the job) if any requested platform is missing in the source.
- Space cleanup (`docker buildx prune`, `docker image prune -a -f`) runs after every image because images up to ~40GB are supported.

## Secrets

- Sync: `ALIYUN_REGISTRY`, `ALIYUN_NAME_SPACE`, `ALIYUN_REGISTRY_USER`, `ALIYUN_REGISTRY_PASSWORD`.
- Build: `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`.

## Gotchas

- Editing `images.txt` on `main` does nothing; commit to `sync`.
- README.md references a workflow file `docker.yaml` that doesn't exist; the real file is `docker-sync.yaml`.
- `dockerfiles/` are plain Dockerfiles (no extension). The `node-docker-*` Dockerfiles must stay installable in both `apt` (Debian slim) and `apk` (Alpine) worlds respectively.
- Keep the git history style: trivial "Update images.txt" commits are the norm.