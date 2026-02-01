# WireGuard Easy

Easy-to-manage VPN server with web UI.

## Access

- **HTTPS (via Caddy)**: https://wireguard.internal.keepdream.in
- **Direct Web UI**: http://192.168.68.73:51831

## Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 51830 | UDP | WireGuard VPN tunnel (external) |
| 51831 | TCP | Web UI (external fallback) |

## Deployment

```powershell
$env:DOCKER_HOST = "npipe:////./pipe/podman-machine-default"
cd Wireguard
particle container push --device 422a0600000000002e257941
```

## Storage

- `/mnt/sdcard/wireguard/` - WireGuard configuration and peer data

## Network

- `wg` network (10.42.42.0/24) - Internal VPN network
- `proxy-network` - For Caddy reverse proxy access

## Notes

- External port forwards (51830/udp, 51831/tcp) remain active for remote VPN access
- The Caddy proxy provides a nicer internal URL but doesn't affect external connectivity
- INSECURE=true is set for local network use
