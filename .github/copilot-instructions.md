# TachyonApps Development Guidelines

This repository contains a collection of containerized applications ("Blueprints") designed to run on Tachyon or Raspberry Pi devices. Each application is managed via a `blueprint.yaml` and a `docker-compose.yml`.

## Architecture & conventions

- **App Structure**: Each application resides in its own top-level directory (e.g., [VeritasKanban](../VeritasKanban)).
- **Blueprint Files**: Every app must have a [blueprint.yaml](../VeritasKanban/blueprint.yaml) file at its root containing metadata like `slug`, `version`, and `containers`.
- **Docker Compose**: The actual deployment logic is in a subdirectory (e.g., [veritas-kanban/docker-compose.yml](../VeritasKanban/veritas-kanban/docker-compose.yml)).
- **Persistent Storage**:
  - All apps MUST store persistent data on the SD card located at `/mnt/sdcard`.
  - Path convention: `/mnt/sdcard/<app-slug>/...`
  - Example: `volumes: [/mnt/sdcard/veritas-kanban/data:/app/data]` in [VeritasKanban/veritas-kanban/docker-compose.yml](../VeritasKanban/veritas-kanban/docker-compose.yml).
- **Mount Dependency**:
  - Apps relying on `/mnt/sdcard` must include a `mount-check` service that waits for the mount point to be ready before starting the main service.
  - Refer to [Wireguard/wg-easy/docker-compose.yml](../Wireguard/wg-easy/docker-compose.yml) for the standard `mount-check` implementation.
- **System Services**: The [Storage/sdcard-mount](../Storage/sdcard-mount) service is responsible for performing the actual mount and should not be modified unless changing system-wide storage logic.

## Developer Workflows

- **Deployment**: Use the `particle` CLI tool to deploy or update containers.
- **Pushing Updates**: Run `particle container push` from within the specific app's root directory (e.g., `cd OpenNotebook; particle container push`).
  - Command: `particle container push --device 422a0600000000002e257941`
  - The tool builds and pushes the application defined in the current directory to the targeted device.
- **Configuration**: Environment-specific device information is stored in `.particle_env.yaml` (git-ignored), which is used by the `particle` tool.

## Device Information
- **Default Device ID**: `422a0600000000002e257941` (Use with `--device` flag in `particle` CLI).

## Key Files to Reference
- [AdGuardHome/blueprint.yaml](../AdGuardHome/blueprint.yaml): Reference for DNS/Network service metadata.
- [Temporal/temporal/docker-compose.yml](../Temporal/temporal/docker-compose.yml): Reference for complex multi-container workflows (though it currently uses anonymous volumes).
- [Storage/sdcard-mount/docker-compose.yml](../Storage/sdcard-mount/docker-compose.yml): Root service for storage management.
