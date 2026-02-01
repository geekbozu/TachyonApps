# TachyonApps

A collection of containerized applications ("Blueprints") specifically designed for Tachyon devices and Raspberry Pi. This repository serves as a home lab source of truth for deploying and managing services with a focus on persistent storage and high availability on edge devices.

## 🏗️ Architecture & Conventions

Each application in this repository follows a standardized structure to ensure consistency across deployments:

- **App Structure**: Every service is contained within its own top-level directory.
- **Blueprint Metadata**: A `blueprint.yaml` file defines the application's metadata (slug, version, categories, etc.).
- **Docker Compose**: Deployment logic resides in a subdirectory (e.g., `AppName/app-name/docker-compose.yml`).
- **Persistent Storage**: 
  - All applications store data on the SD card located at `/mnt/sdcard`.
  - Data path convention: `/mnt/sdcard/<app-slug>/`
- **Mount Dependency**: Services that rely on the SD card include a `mount-check` sidecar that ensures the storage is ready before the main application starts.

## 🚀 Deployment

Applications are managed using the `particle` CLI tool.

### Pushing Updates
To build and deploy a container to a specific device:

1. Navigate to the application's root directory:
   ```bash
   cd VeritasKanban
   ```
2. Push to the device:
   ```bash
   particle container push --device <YOUR_DEVICE_ID>
   ```

### Configuration
Device-specific information is stored in a local `.particle_env.yaml` file. This file is excluded from version control to prevent sensitive data from being shared.

## 📦 Available Applications

- **[AdGuard Home](./AdGuardHome)**: Network-wide ads & trackers blocking DNS server with auto-provisioned DNS rewrites.
- **[Caddy](./Caddy)**: Reverse proxy with automatic HTTPS for all local services.
- **[OpenNotebook](./OpenNotebook)**: Collaborative notebook environment with SurrealDB.
- **[Veritas Kanban](./VeritasKanban)**: Lightweight Kanban board with AI-ready API support.
- **[Wireguard](./Wireguard)**: Easy-to-manage VPN server.
- **[Storage](./Storage)**: System-level service for SD card auto-mounting.
- **[Temporal](./Temporal)**: Workflow orchestration engine (standalone, not behind reverse proxy).

## 🌐 Reverse Proxy (Caddy)

All services (except Temporal) are accessible via HTTPS through Caddy reverse proxy:

| Service | URL |
|---------|-----|
| Veritas Kanban | `https://tachyon.local` |
| AdGuard Home | `https://adguard.tachyon.local` |
| WireGuard | `https://wireguard.tachyon.local` |
| OpenNotebook | `https://notebook.tachyon.local` |
| OpenNotebook API | `https://notebook-api.tachyon.local` |

DNS rewrites for `*.tachyon.local` are auto-provisioned by AdGuard Home.

## 🛠️ Development

### Podman Support
This repository uses Podman. Set the Docker host before running particle commands:
```powershell
$env:DOCKER_HOST = "npipe:////./pipe/podman-machine-default"
```

### Local Environment
- Ensure you have the `particle` CLI tool installed.
- Maintain the `/mnt/sdcard` directory structure on your target device to ensure persistence.
- Refer to the `blueprint.yaml` in each directory for specific container dependencies.

---
*Note: This repository is intended for personal use and home lab reference.*
