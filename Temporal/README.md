# Temporal Deployment for Particle Tachyon

This deployment uses SD card storage for persistent configuration while keeping PostgreSQL data on container-local storage for performance and reliability.

## Prerequisites

1. **SD Card Mount**: Ensure the `Storage/sdcard-mount` container is running to mount the SD card at `/mnt/sdcard`

## Setup Instructions

### 1. Prepare SD Card Configuration

Before starting the Temporal containers, copy the configuration file to the SD card:

```bash
# Create directory structure on SD card
mkdir -p /mnt/sdcard/temporal/dynamicconfig

# Copy dynamic configuration
cp temporal/dynamicconfig/development-sql.yaml /mnt/sdcard/temporal/dynamicconfig/
```

### 2. Start Services

```bash
cd temporal
docker compose up -d
```

## Storage Architecture

### SD Card Storage (`/mnt/sdcard/temporal/`)
- **dynamicconfig/**: Temporal runtime configuration

This configuration persists across container recreations and firmware updates.

### Container-Local Storage
- **PostgreSQL data** (`temporal-postgres-data` named volume): Stored in Docker's volume directory for performance
  - ⚠️ **Important**: PostgreSQL data is NOT on SD card to avoid performance issues and potential corruption
  - Data persists across container recreations but not firmware updates
  - Use `docker volume ls` to see the volume and `docker volume inspect temporal-postgres-data` for details

### Embedded Initialization Scripts
- **Database setup**: Embedded directly in the `temporal-admin-tools` service
- **Namespace creation**: Embedded directly in the `temporal-create-namespace` service
- No manual script copying required - these run automatically on first startup

## Service Dependencies

1. `sdcard-mount`: Mounts and ensures SD card availability (used in consolidated `tachyon-stack`)
2. `postgresql`: Database with health checks
3. `temporal-admin-tools`: Initializes database schemas using embedded scripts (depends on postgresql)
4. `temporal`: Main server (depends on admin-tools completion)
5. `temporal-create-namespace`: Creates default namespace using embedded script (depends on temporal health)
6. `temporal-ui`: Web interface (depends on temporal health)

Note: Per-app `mount-check` sidecars were removed in favor of the central `sdcard-mount` service when using the consolidated stack.

## Ports

- `5432`: PostgreSQL
- `7233`: Temporal gRPC API
- `8080`: Temporal Web UI

## Backup Recommendations

Since PostgreSQL data is not on the SD card:
- Use regular database backups via `pg_dump`
- Consider backing up to SD card: `/mnt/sdcard/temporal/backups/`

Example backup script:
```bash
docker exec temporal-postgresql pg_dumpall -U temporal > /mnt/sdcard/temporal/backups/backup-$(date +%Y%m%d-%H%M%S).sql
```
