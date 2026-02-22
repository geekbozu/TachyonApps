# TachyonApps Development Guidelines

This repository contains a collection of containerized applications deployed as a single Docker Compose stack on Tachyon or Raspberry Pi devices. A **Bootstrap** blueprint is pushed once via `particle container push`; it clones the repo to the host and brings up the entire stack.

## Architecture & conventions

- **Single Stack**: A top-level `docker-compose.yml` uses `include` directives to compose each app's compose file. Startup order is enforced:
  `sdcard-mount (healthy) → adguardhome & caddy → remaining apps`.
- **App Structure**: Each application resides in its own top-level directory (e.g., `AdGuardHome/`).
- **Docker Compose**: The actual deployment logic is in a subdirectory (e.g., `AdGuardHome/adguardhome/docker-compose.yml`).
- **Blueprint**: Only `Bootstrap/blueprint.yaml` exists. Individual app blueprints have been removed.
- **Persistent Storage**:
  - All apps MUST store persistent data on the SD card located at `/mnt/sdcard`.
  - Path convention: `/mnt/sdcard/<app-slug>/...`
- **Mount Dependency**:
  - The `sdcard-mount` service (in `Storage/sdcard-mount/docker-compose.yml`) has a Docker healthcheck.
  - Apps that need `/mnt/sdcard` declare `depends_on: sdcard-mount: condition: service_healthy`.
  - **No per-app `mount-check` sidecars** — the healthcheck on `sdcard-mount` replaces them.
- **Shared Network**: A single `proxy-network` is defined in the top-level compose. Sub-files reference it without declaring it.
- **System Services**: The `Storage/sdcard-mount` service is responsible for performing the actual mount and should not be modified unless changing system-wide storage logic.

## Developer Workflows

- **Deployment**: Push the **Bootstrap** container only — it handles deploying the full stack.
  - Command: `cd Bootstrap; particle container push --device 422a0600000000002e257941`
  - The bootstrap container clones/pulls the repo to `/home/particle/remote_stack` and runs `docker compose up -d` as the `particle` user via `nsenter`.
  - Supports submodules (`Flux/flux-repo`, `Pop3Sync/pop3-gmail-importer-upstream`) — they are auto-initialized.
- **Branch Selection**: Set `BRANCH` env var to deploy a non-default branch (default: `MergeItAll`).
  - Example: `BRANCH=feature/foo particle container push --device 422a0600000000002e257941`
- **Updating**: Re-push Bootstrap. It will `git pull` latest changes and re-run compose up.
- **Configuration**: Environment-specific device information is stored in `.particle_env.yaml` (git-ignored), which is used by the `particle` tool.

## Device Information
- **Default Device ID**: `422a0600000000002e257941` (Use with `--device` flag in `particle` CLI).

## Key Files to Reference
- `Bootstrap/blueprint.yaml`: The single entry-point blueprint.
- `docker-compose.yml` (root): Top-level compose that includes all app compose files.
- `Storage/sdcard-mount/docker-compose.yml`: Root service for storage management with healthcheck.
- `Caddy/caddy/docker-compose.yml`: Reverse proxy configuration.
