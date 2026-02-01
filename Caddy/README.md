# Caddy Reverse Proxy

HTTPS reverse proxy for all TachyonApps services using self-signed certificates for `.local` domains.

## Services Proxied

| Internal Service | External URL | Port |
|-----------------|--------------|------|
| veritas-kanban:3001 | https://tachyon.local | 443 |
| adguardhome:3000 | https://adguard.tachyon.local | 443 |
| wg-easy:51821 | https://wireguard.tachyon.local | 443 |
| open-notebook:8502 | https://notebook.tachyon.local | 443 |
| open-notebook:5055 | https://notebook-api.tachyon.local | 443 |

## Configuration

The Caddyfile is generated inline by the `caddy-config` init container and written to `/mnt/sdcard/caddy/etc/Caddyfile`.

## Network

All proxied services must join the `proxy-network` Docker network to be reachable by Caddy.

## Deployment

```powershell
$env:DOCKER_HOST = "npipe:////./pipe/podman-machine-default"
cd Caddy
particle container push --device <YOUR_DEVICE_ID>
```

## Storage

- `/mnt/sdcard/caddy/etc/` - Caddyfile (generated)
- `/mnt/sdcard/caddy/data/` - Certificates and persistent data
- `/mnt/sdcard/caddy/config/` - Caddy config state
