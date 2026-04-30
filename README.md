# OpenClaw Docker Auto-Build

This repository builds OpenClaw Docker images and publishes them to GHCR.

Workflow file: `.github/workflows/check-and-build.yml`
Workflow name: `Build Patched OpenClaw`

## Build Flow (Current Behavior)

The workflow runs every 4 hours and can also be started manually.

1. Resolve version tag to build
- Uses manual `version` input if provided.
- Otherwise resolves the latest stable upstream package tag from `ghcr.io/openclaw/openclaw`.
- Falls back to `openclaw/openclaw` git tags via GitHub API if GHCR lookup fails.

2. Validate tag safety
- Blocks pre-release tags (`alpha`, `beta`, `rc`, `preview`).
- Requires tag format matching `^v?[0-9]+(\.[0-9]+){1,3}$`.

3. Resolve checkout ref
- Checks upstream git tags for both forms (`v1.2.3` and `1.2.3`) and uses the first match.

4. Check existing images in GHCR
- Checks `full` (`<tag>`) and `slim` (`<tag>-slim`).
- Also checks alternate tag form with/without `v`.
- For runs that should update latest aliases, also verifies `latest` and `latest-slim` presence and digest match.
- If `force_rebuild=true`, skips existence checks and rebuilds both variants.

5. Build only missing variants
- Full variant build args:
  - `OPENCLAW_INSTALL_BROWSER=1`
  - `OPENCLAW_DOCKER_APT_PACKAGES=socat`
- Slim variant build args:
  - `OPENCLAW_INSTALL_BROWSER=0`

6. Optional post-build 1Password CLI layer
- Unless `skip_1password=true`, adds a second layer that installs `1password-cli`.
- Applied to each variant that was built in this run.

7. Output summary and notifications
- Writes image details to GitHub Step Summary.
- Sends Slack success/failure notifications when `SLACK_WEBHOOK_URL` is set and at least one variant was built.

## Published Image Variants

- Full image: `<version-tag>`
- Slim image: `<version-tag>-slim`

Image name is based on repository owner and name:

- `ghcr.io/${{ github.repository_owner }}/${{ github.event.repository.name }}:<tag>`

For this repo:

- `ghcr.io/ioleksiy/openclaw-dckr:<tag>`

## Tag Behavior

- Version tags are published for each variant that is built.
- Scheduled runs always include latest aliases:
  - `latest` (full)
  - `latest-slim` (slim)
- Manual runs include latest aliases only if `isLatest=true`.
- Existing tags are skipped unless rebuild is required or `force_rebuild=true`.

## Manual Run Inputs

From the Actions UI, open `Build Patched OpenClaw` and provide optional inputs:

- `version` (string): specific tag (example `v1.0.0`). Empty means latest stable upstream.
- `isLatest` (boolean, default `false`): also publish `latest` and `latest-slim` for this manual run.
- `force_rebuild` (boolean, default `false`): build even if tags already exist.
- `skip_1password` (boolean, default `false`): skip the 1Password CLI layering steps.

## Permissions and Auth

- `contents: read`
- `packages: write`
- Uses `GITHUB_TOKEN` for GHCR auth and API calls.
