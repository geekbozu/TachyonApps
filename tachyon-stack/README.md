# Tachyon Stack (consolidated)

This folder contains a consolidated Docker Compose stack that brings up the SD card mount service (`sdcard-mount`) and Caddy first, then AdGuard, then the rest of the apps.

Quick start:

1. Review `docker-compose.yml` to confirm device path (defaults to `/dev/mmcblk1p1`).
2. Start the stack:

```bash
cd tachyon-stack
docker compose up -d
```

Notes:
- `sdcard-mount` includes a healthcheck; services that rely on the SD card are configured to wait for `sdcard-mount` to be healthy.
- Per-app `mount-check` helper services have been removed in favor of the central `sdcard-mount` service.
- Keep existing per-app blueprints and configuration files in their original locations; the consolidated stack references them for builds/volumes.

If you want `sdcard-mount` to use a different device, edit `docker-compose.yml` and change the `DEVICE` variable in the `sdcard-mount` command.
