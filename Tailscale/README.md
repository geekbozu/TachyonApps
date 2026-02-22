# Tailscale Exit Node

Configures the device as a Tailscale exit node and subnet router, connecting it to a custom Headscale server.

## Features

- Connects to custom Headscale server at `https://mail.keepdream.in:18080`
- Advertises the device as an exit node (`--advertise-exit-node`)
- Advertises the local subnet `192.168.68.0/24` (`--advertise-routes=192.168.68.0/24`)
- Ignores Tailscale DNS settings to allow local DNS resolution (`--accept-dns=false`)
- Assigns operator permissions to the `geek` user (`--operator=geek`)

## Configuration

The container expects a pre-shared authentication key to be available at:
`/mnt/sdcard/tailscale/key/preauth.key.save`

This key is read on startup and used to authenticate the **host device's** Tailscale installation with the Headscale server.

## Storage

- `/mnt/sdcard/tailscale/key/` - Contains the pre-shared auth key

## Deployment

```powershell
$env:DOCKER_HOST = "npipe:////./pipe/podman-machine-default"
cd Tailscale
particle container push --device <YOUR_DEVICE_ID>
```

## Troubleshooting

**Device not showing up in Headscale:**
- Verify the pre-shared key exists at `/mnt/sdcard/tailscale/key/preauth.key.save`
- Check the container logs: `docker logs -f tailscale`
- Ensure the Headscale server is reachable from the device

**Routes not working:**
- Ensure IP forwarding is enabled on the host device
- Check if the routes need to be approved in the Headscale admin interface
