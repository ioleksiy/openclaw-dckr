# OpenClaw Docker Auto-Build

This repository automatically builds OpenClaw Docker images and publishes them to GHCR.

The workflow follows the standard upstream OpenClaw Docker build process, then adds a small set of practical defaults for container usage.

## What this repo does

Workflow: `.github/workflows/check-and-build.yml`

- Checks upstream OpenClaw container package tags every 4 hours.
- Picks the latest stable tag from `ghcr.io/openclaw/openclaw`.
- Skips beta/pre-release package tags (based on the actual image tag that would be published).
- Skips building if that tag already exists in GHCR.
- Builds and pushes only when a new image is needed.

## Standard build, plus small additions

The image is built from upstream OpenClaw source and Docker setup, with these extra build args:

- `OPENCLAW_INSTALL_BROWSER=1`
  - Includes browser dependencies in the image.
- `OPENCLAW_DOCKER_APT_PACKAGES=socat`
  - Adds `socat`, which is useful in many container networking and forwarding scenarios.

## Image naming

Image name is automatic and based on owner + repository name:

- `ghcr.io/${{ github.repository_owner }}/${{ github.event.repository.name }}:<version-tag>`

For this repository, it becomes:

- `ghcr.io/ioleksiy/openclaw-dckr:<version-tag>`

## Tags behavior

- Every successful build publishes the version tag (for example `v1.2.3`).
- Only scheduled automatic builds also publish `latest`.
- Manual runs do not move `latest`.

## Manual run options

You can run the workflow from the Actions UI with inputs:

- `version` (optional): specific tag to build (example: `v1.0.0`).
  - If empty, it resolves to the latest stable upstream package tag.
- `force_rebuild` (optional, default `false`): build even if image tag already exists.

Manual run steps:

1. Open **Actions**.
2. Select **Build Custom OpenClaw**.
3. Click **Run workflow**.
4. Fill inputs as needed.
5. Start the run.

## Safety checks

- Beta tags are blocked.
- Beta/pre-release detection is done on the final package/image tag, not only release metadata.
- Tag format is validated before checkout/build.
- Existing tags are checked in GHCR to avoid duplicate work (unless force rebuild is enabled).

## Permissions and auth

- Requires `packages: write` permission.
- Uses GitHub-provided `GITHUB_TOKEN` to authenticate to GHCR.
