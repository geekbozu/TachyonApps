# Veritas Kanban Deployment

This application uses a git submodule for the source code and requires special build steps.

## Prerequisites

- Podman must be running (we use Podman instead of Docker)
- The `veritas-kanban-repo` submodule must be initialized and checked out to v1.1.0

## Initial Setup

```powershell
# Initialize and update submodule (if not already done)
cd C:\Users\geek\Documents\Repos\TachyonApps
git submodule update --init VeritasKanban/veritas-kanban-repo

# Checkout the stable release
cd VeritasKanban/veritas-kanban-repo
git checkout v1.1.0
```

## Environment Configuration

Create the `.env` file on the device at `/mnt/sdcard/veritas-kanban/.env`:

```bash
# Node Environment
NODE_ENV=production
PORT=3001

# Data Storage
DATA_DIR=/app/data

# Authentication
VERITAS_AUTH_ENABLED=false
VERITAS_ADMIN_KEY=your-secure-admin-key-here-min-32-chars

# Optional: JWT Secret (auto-generated if not provided)
#VERITAS_JWT_SECRET=your-jwt-secret-here

# Optional: CORS Configuration (for remote access)
#CORS_ORIGINS=https://kanban.example.com
```

To provision via SSH:

```bash
ssh user@your-device
sudo mkdir -p /mnt/sdcard/veritas-kanban/data
sudo nano /mnt/sdcard/veritas-kanban/.env
# Paste the content above
sudo chmod 600 /mnt/sdcard/veritas-kanban/.env
```

## Building and Deploying

**Important**: Due to Podman usage, you must set the `DOCKER_HOST` environment variable before running the particle command.

```powershell
# Navigate to the VeritasKanban directory
cd C:\Users\geek\Documents\Repos\TachyonApps\VeritasKanban

# Build and push with Podman (no user prompts)
$env:DOCKER_HOST = "npipe:////./pipe/podman-machine-default"
particle container push --device 422a0600000000002e257941
```

Or as a one-liner:

```powershell
cd C:\Users\geek\Documents\Repos\TachyonApps\VeritasKanban; $env:DOCKER_HOST = "npipe:////./pipe/podman-machine-default"; particle container push --device 422a0600000000002e257941
```

## Updating to New Version

```powershell
# Update the submodule
cd C:\Users\geek\Documents\Repos\TachyonApps\VeritasKanban\veritas-kanban-repo
git fetch --tags
git checkout v1.x.x  # Replace with desired version

# Then run the build and deploy command above
```

## Troubleshooting

- **"blueprint.yaml not found"**: Make sure you're in the `VeritasKanban` directory, not `TachyonApps`
- **Docker connection errors**: Ensure Podman is running and `DOCKER_HOST` is set
- **Build failures**: The Dockerfile has been modified to use `--no-frozen-lockfile --ignore-scripts` for the production install step

## Notes

- This app requires a custom Dockerfile modification to build successfully (lockfile workaround)
- The submodule modifications are tracked separately from the main repo
- Authentication is disabled for internal POC use, but the admin key is still required by the application

## Access URLs

- **HTTPS (via Caddy)**: https://tachyon.local
- **Direct**: http://192.168.68.73:3001

## Network

Connected to `proxy-network` for Caddy reverse proxy access.

## Storage

- `/mnt/sdcard/veritas-kanban/data/` - Application data
- `/mnt/sdcard/veritas-kanban/.veritas-kanban/` - App config
- `/mnt/sdcard/veritas-kanban/.env` - Environment variables
