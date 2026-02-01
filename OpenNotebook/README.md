# OpenNotebook

Collaborative AI notebook environment with SurrealDB backend.

## Access

- **HTTPS (via Caddy)**: https://notebook.internal.keepdream.in
- **API (via Caddy)**: https://notebook-api.internal.keepdream.in
- **Direct Web UI**: http://192.168.68.73:8502
- **Direct API**: http://192.168.68.73:5055

## Configuration

Environment variables are stored in `/mnt/sdcard/open-notebook/.env` on the device.

### Required Environment Variables

Create `/mnt/sdcard/open-notebook/.env`:
```env
SURREAL_USER=root
SURREAL_PASSWORD=your-secure-password
# Add additional OpenNotebook configuration here
```

## Deployment

```powershell
$env:DOCKER_HOST = "npipe:////./pipe/podman-machine-default"
cd OpenNotebook
particle container push --device 422a0600000000002e257941
```

## Storage

- `/mnt/sdcard/open-notebook/surreal_data/` - SurrealDB database files
- `/mnt/sdcard/open-notebook/notebook_data/` - OpenNotebook app data
- `/mnt/sdcard/open-notebook/.env` - Environment variables

## Services

- `surrealdb` - Database backend (port 8000 internal)
- `open_notebook` - Web UI and API

## Network

Connected to `proxy-network` for Caddy reverse proxy access.
