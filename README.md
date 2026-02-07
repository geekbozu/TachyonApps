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
- **Mount Dependency**: The SD card is managed by a central `sdcard-mount` service in the consolidated stack (`tachyon-stack`). Per-app `mount-check` sidecars were removed in favor of the central service. Use `sdcard-mount` when deploying the consolidated stack.

## 🚀 Deployment

Applications are managed using the `particle` CLI tool.

### Pushing Updates
To build and deploy a container to a specific device:

1. Navigate to the application's root directory:
   ```bash
   cd AdGuardHome
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
- **[Wireguard](./Wireguard)**: Easy-to-manage VPN server.
- **[Storage](./Storage)**: System-level service for SD card auto-mounting.
- **[Temporal](./Temporal)**: Workflow orchestration engine (standalone, not behind reverse proxy).

## 🌐 Reverse Proxy (Caddy)

All services (except Temporal) are accessible via HTTPS through Caddy reverse proxy:

| Service | URL |
|---------|-----|
| AdGuard Home | `https://adguard.internal.keepdream.in` |
| WireGuard | `https://wireguard.internal.keepdream.in` |
| OpenNotebook | `https://notebook.internal.keepdream.in` |
| OpenNotebook API | `https://notebook-api.internal.keepdream.in` |

### DNS & Certificate Setup

The infrastructure uses Let's Encrypt wildcard certificates with DNS-01 validation via Porkbun:

1. **External DNS (Porkbun)**:
   - `*.internal.keepdream.in` → Your home/WireGuard IP
   - `internal.keepdream.in` → Same IP

2. **Internal DNS (AdGuard Home)**:
   - `*.internal.keepdream.in` → Local Tachyon IP (192.168.68.73)
   - Auto-provisioned on AdGuard deployment

3. **Certificate Management**:
   - Wildcard cert covers all `*.internal.keepdream.in` subdomains
   - Currently using **Let's Encrypt STAGING** for testing
   - Switch to production by removing `acme_ca` line in Caddy config
   - DNS-01 challenge requires Porkbun API credentials (no port forwarding needed)

This setup ensures services are only accessible via WireGuard while using valid certificates.

### Required Configuration

Create `/mnt/sdcard/caddy/.env` on the device:
```env
PORKBUN_API_KEY=pk1_your_api_key
PORKBUN_SECRET_KEY=sk1_your_secret_key  
LETSENCRYPT_EMAIL=your-email@example.com
```

## 🛠️ Development

### Podman Support
This repository uses Podman. Set the Docker host before running particle commands:
```powershell
$env:DOCKER_HOST = "npipe:\\\\.\pipe\podman-machine-default"
```

### Local Environment
- Ensure you have the `particle` CLI tool installed.
- Maintain the `/mnt/sdcard` directory structure on your target device to ensure persistence.
- Refer to the `blueprint.yaml` in each directory for specific container dependencies.

---
*Note: This repository is intended for personal use and home lab reference.*
