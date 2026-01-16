# TachyonApps Repository - Copilot Instructions

This repository contains containerized applications designed for the Tachyon device and Raspberry Pi platforms. Applications are packaged with Docker Compose configurations and blueprint.yaml metadata files for the Tachyon app ecosystem.

## Repository Structure

- `AdGuardHome/`: Network-wide DNS-based ad blocker application
- `Storage/`: SD card auto-mount utility for persistent storage
- `Wireguard/`: WireGuard VPN server with web UI (wg-easy)
- Each application directory contains:
  - `blueprint.yaml`: Application metadata and configuration
  - Subdirectory with `docker-compose.yml`: Container orchestration configuration
  - Optional `.particle_env.yaml`: Environment-specific settings

## Code Standards

### Blueprint YAML Files

Blueprint files define application metadata and must follow this structure:
- `slug`: Unique identifier (kebab-case)
- `type`: Application category (e.g., DNS, VPN, System)
- `category`: Display category for app store
- `expertiseLevel`: Required user expertise (BEGINNER, INTERMEDIATE, ADVANCED, EXPERT)
- `tags`: Array of descriptive tags
- `gitrepo`: Repository URL
- `name`: Display name
- `shortDescription`: Brief one-line description
- `version`: Semantic versioning (e.g., 1.0.0)
- `supportedDevices`: List devices (Tachyon, Raspberry Pi)
- `containers`: Container name(s)
- `introduction`: Brief introduction text
- `description`: Detailed markdown description with features list

### Docker Compose Configurations

1. **Volume Management**
   - Use `/mnt/sdcard/[app-name]` for persistent configuration storage
   - Mount SD card as read-only (`:ro`) for verification containers
   - Use named volumes for temporary/cache data

2. **Container Naming**
   - Use `container_name` field for clear identification
   - Prefix helper containers with app name (e.g., `adguardhome-mount-check`)

3. **SD Card Mount Dependency Pattern**
   - Include a `mount-check` service before main application starts
   - Use Alpine Linux for lightweight verification
   - Verify mount with `mountpoint -q /mnt/sdcard` check
   - Retry logic with timeout (typically 5 attempts, 5-second intervals)
   - Use `depends_on` with `condition: service_completed_successfully`

4. **Network Configuration**
   - Use `network_mode: host` when container needs direct host network access
   - Define custom bridge networks when isolation is needed
   - Document exposed ports with comments for optional services

5. **Privilege and Capabilities**
   - Use `privileged: true` only when necessary (e.g., mounting, network configuration)
   - Prefer specific `cap_add` capabilities over privileged mode when possible
   - Document why elevated privileges are required

6. **Restart Policies**
   - Use `restart: unless-stopped` for main services
   - Use `restart: "no"` for one-time initialization containers
   - Helper containers should exit after completing their task

7. **Shell Commands in Docker Compose**
   - Use `sh -c 'script'` for inline shell scripts
   - Escape dollar signs with `$$` in compose YAML
   - Include echo statements for debugging/logging
   - Add proper error handling with `|| exit 1`
   - For long-running services, use infinite loops with sleep intervals

### Environment Variables

- Set timezone with `TZ` environment variable (default: UTC)
- Use `.particle_env.yaml` for device-specific configuration
- Document required vs optional environment variables

## Testing and Validation

Since this repository contains Docker Compose configurations rather than compiled code:

1. **Validation Steps**
   - Verify YAML syntax with `docker-compose config` for each application
   - Check that all referenced paths exist or are documented as requirements
   - Ensure SD card mount paths are consistent across applications
   - Validate that port mappings don't conflict between applications

2. **Manual Testing**
   - Test container startup sequence with `docker-compose up`
   - Verify mount-check containers succeed before main services start
   - Check container logs for proper initialization
   - Ensure web UIs are accessible on documented ports

3. **No automated test framework** is currently in place for this repository

## Device Compatibility

- **Tachyon**: Primary target device, ARM-based
- **Raspberry Pi**: Secondary supported platform
- Applications must work on ARM architecture
- Consider SD card storage limitations
- Plan for firmware re-install scenarios (data persists on SD card)

## Key Guidelines

1. **Minimal Changes**: Make surgical updates to existing configurations
2. **SD Card Dependency**: Most apps require persistent storage on `/mnt/sdcard`
3. **Documentation**: Update blueprint.yaml descriptions when changing features
4. **Port Conflicts**: Be aware of common ports (53 for DNS, 51830/51831 for VPN, etc.)
5. **SystemD Integration**: Some containers interact with host systemd services (e.g., systemd-resolved)
6. **Security**: Applications run in containers but may need elevated privileges for network/storage operations

## Common Patterns

### Adding a New Application

1. Create new directory under repository root with descriptive name
2. Create `blueprint.yaml` with complete metadata
3. Create subdirectory with application name containing `docker-compose.yml`
4. Include mount-check pattern if SD card storage is needed
5. Document all exposed ports and required configurations
6. Test on target hardware before committing

### Modifying Existing Applications

1. Test configuration changes with `docker-compose config`
2. Update `version` field in blueprint.yaml for significant changes
3. Update `description` if features change
4. Ensure backward compatibility with existing deployments
5. Document breaking changes clearly
