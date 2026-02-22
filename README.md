# TachyonApps

A collection of containerized applications deployed as a single stack on Tachyon devices and Raspberry Pi. A **Bootstrap** blueprint clones this repository to the device and brings up the entire Docker Compose stack with one push.

## 🏗️ Architecture & Conventions

All services are orchestrated through a single top-level `docker-compose.yml` that uses `include` directives to compose each app's compose file. Startup order is strictly enforced via healthchecks and `depends_on`:

```
sdcard-mount (healthy) → adguardhome & caddy → remaining apps
```

- **App Structure**: Every service is contained within its own top-level directory.
- **Docker Compose**: Deployment logic resides in a subdirectory (e.g., `AppName/app-name/docker-compose.yml`).
- **Persistent Storage**: 
  - All applications store data on the SD card located at `/mnt/sdcard`.
  - Data path convention: `/mnt/sdcard/<app-slug>/`
- **Mount Dependency**: The `sdcard-mount` service has a Docker healthcheck. All apps that need `/mnt/sdcard` declare `depends_on: sdcard-mount: condition: service_healthy` — no per-app `mount-check` sidecars are needed.
- **Shared Network**: A single `proxy-network` is defined in the top-level compose and shared by all services that sit behind Caddy.

## 🚀 Deployment

The entire stack is deployed through the **Bootstrap** blueprint.

### First-Time Setup
1. Navigate to the Bootstrap directory:
   ```bash
   cd Bootstrap
   ```
2. Push the bootstrap container to the device:
   ```bash
   particle container push --device <YOUR_DEVICE_ID>
   ```

The bootstrap container will:
1. Clone this repository to `/home/particle/remote_stack` on the host (with submodules).
2. Set ownership to the `particle` user.
3. Run `docker compose up -d` from the cloned repo via `nsenter` as the `particle` user.

### Branch Selection
By default the bootstrap deploys from the `MergeItAll` branch. Override with the `BRANCH` env var:
```bash
BRANCH=main particle container push --device <YOUR_DEVICE_ID>
```

### Updating
Push the Bootstrap container again. It will `git pull` the latest changes and re-run `docker compose up -d`.

### First-Time Cleanup
If migrating from the old per-app deployment model, clean up stale containers and networks on the device before pushing Bootstrap:
```bash
docker rm -f caddy-config caddy caddy-mount-check \
  adguardhome adguardhome-resolved-fix adguardhome-mount-check adguardhome-dns-provision \
  flux-web flux-mount-check \
  open-notebook open-notebook-mount-check surrealdb \
  pop3-gmail-importer pop3-gmail-importer-mount-check \
  wg-easy wg-easy-mount-check \
  tailscale-host-config tailscale-mount-check \
  sdcard-mount 2>/dev/null
docker network rm proxy-network 2>/dev/null
```

### Configuration
Device-specific information is stored in a local `.particle_env.yaml` file (git-ignored).

## 📦 Available Applications

- **[AdGuard Home](./AdGuardHome)**: Network-wide ads & trackers blocking DNS server with auto-provisioned DNS rewrites.
- **[Caddy](./Caddy)**: Reverse proxy with automatic HTTPS for all local services.
- **[Flux](./Flux)**: AI-powered Kanban board with MCP integration.
- **[OpenNotebook](./OpenNotebook)**: AI-powered notebook with SurrealDB.
- **[Pop3Sync](./Pop3Sync)**: POP3 → Gmail importer.
- **[Wireguard](./Wireguard)**: Easy-to-manage VPN server.
- **[Tailscale](./Tailscale)**: Tailscale exit node & subnet router.
- **[Storage](./Storage)**: System-level service for SD card auto-mounting.
- **[Temporal](./Temporal)**: Workflow orchestration engine (currently disabled — uncomment in `docker-compose.yml` when fixed).

## 🌐 Reverse Proxy (Caddy)

All services (except Temporal) are accessible via HTTPS through Caddy reverse proxy:

| Service | URL |
|---------|-----|
| AdGuard Home | `https://adguard.internal.keepdream.in` |
| WireGuard | `https://wireguard.internal.keepdream.in` |
| OpenNotebook | `https://notebook.internal.keepdream.in` |
| OpenNotebook API | `https://notebook-api.internal.keepdream.in` |
| Flux | `https://flux.internal.keepdream.in` |

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

### Repository Structure
```
Bootstrap/           # Single entry-point blueprint
  blueprint.yaml
  bootstrap/docker-compose.yml
docker-compose.yml   # Top-level compose (include all apps)
AdGuardHome/         # DNS ad-blocking
Caddy/               # Reverse proxy
Flux/                # Task management
OpenNotebook/        # AI notebook
Pop3Sync/            # Email importer
Storage/             # SD card mount service
Tailscale/           # VPN exit node
Temporal/            # Workflow engine (disabled)
Wireguard/           # VPN server
```

### Podman Support
This repository uses Podman. Set the Docker host before running particle commands:
```powershell
$env:DOCKER_HOST = "npipe:\\\\.\pipe\podman-machine-default"
```

### Local Environment
- Ensure you have the `particle` CLI tool installed.
- Maintain the `/mnt/sdcard` directory structure on your target device to ensure persistence.

---
*Note: This repository is intended for personal use and home lab reference.*
