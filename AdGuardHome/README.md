# AdGuard Home

Network-wide ads & trackers blocking DNS server with web UI.

## Access

- **HTTPS (via Caddy)**: https://adguard.internal.keepdream.in
- **Direct**: http://192.168.68.73:3000

## DNS Auto-Provisioning

The `dns-provision` init container automatically adds DNS rewrites for all TachyonApps services:

- `*.internal.keepdream.in` → 192.168.68.73 (local Tachyon IP)
- `*.tachyon.local` → 192.168.68.73 (legacy support)
- `tachyon.local` → 192.168.68.73

This enables:
1. Internal DNS resolution for services behind WireGuard
2. Valid Let's Encrypt certificates via DNS-01 challenge
3. Traffic stays on local network while using public domain names

## Deployment

```powershell
$env:DOCKER_HOST = "npipe:////./pipe/podman-machine-default"
cd AdGuardHome
particle container push --device 422a0600000000002e257941
```

## Storage

- `/mnt/sdcard/adguardhome/` - AdGuard Home configuration (AdGuardHome.yaml)
- Anonymous volume for work data

## Network

- Port 53 (TCP/UDP) - DNS
- Port 3000 - Web UI
- Connected to `proxy-network` for Caddy reverse proxy

## Init Containers

1. `resolved-fix` - Disables systemd-resolved conflict on port 53
2. `dns-provision` - Auto-provisions DNS rewrites using yq

Note: In the consolidated `tachyon-stack` deployment, SD card availability is provided by the central `sdcard-mount` service. Per-app `mount-check` sidecars were removed.
