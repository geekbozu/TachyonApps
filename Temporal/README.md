# Temporal Deployment for Particle Tachyon

This deployment uses SD card storage for persistent configuration while keeping PostgreSQL data on container-local storage for performance and reliability.

## Prerequisites

1. **SD Card Mount**: Ensure the `Storage/sdcard-mount` container is running to mount the SD card at `/mnt/sdcard`

## Setup Instructions

### 1. Prepare SD Card Directories

Before starting the Temporal containers, copy the configuration files to the SD card:

```bash
# Create directory structure on SD card
mkdir -p /mnt/sdcard/temporal/scripts
mkdir -p /mnt/sdcard/temporal/dynamicconfig

# Copy scripts
cp temporal/scripts/setup-postgres.sh /mnt/sdcard/temporal/scripts/
cp temporal/scripts/create-namespace.sh /mnt/sdcard/temporal/scripts/

# Copy dynamic configuration
cp temporal/dynamicconfig/development-sql.yaml /mnt/sdcard/temporal/dynamicconfig/

# Ensure scripts are executable
chmod +x /mnt/sdcard/temporal/scripts/*.sh
```

### 2. Start Services

```bash
cd temporal
docker compose up -d
```

## Storage Architecture

### SD Card Storage (`/mnt/sdcard/temporal/`)
- **scripts/**: Database initialization and namespace creation scripts
- **dynamicconfig/**: Temporal runtime configuration

These files persist across container recreations and firmware updates.

### Container-Local Storage
- **PostgreSQL data** (`/var/lib/postgresql/data`): Stored in container volume for performance
  - ⚠️ **Important**: PostgreSQL data is NOT on SD card to avoid performance issues and potential corruption

## Service Dependencies

1. `mount-check`: Validates SD card is mounted before starting
2. `postgresql`: Database with health checks
3. `temporal-admin-tools`: Initializes database schemas (depends on mount-check and postgresql)
4. `temporal`: Main server (depends on admin-tools completion)
5. `temporal-create-namespace`: Creates default namespace (depends on temporal health)
6. `temporal-ui`: Web interface (depends on temporal health)

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
