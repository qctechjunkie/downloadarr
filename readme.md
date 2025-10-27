# Downloadarr Media Stack

A complete Docker Compose stack for automated media downloading and management using the *arr suite of applications.

## Overview

This stack provides a complete solution for:
- **Movie Management** (Radarr)
- **Music Management** (Lidarr + Huntarr) 
- **Torrent Downloading** (qBittorrent)
- **Indexer Management** (Prowlarr)
- **CloudFlare Bypass** (FlareSolverr)

## Services

| Service | Port | Purpose |
|---------|------|---------|
| **Radarr** | 7878 | Movie collection manager - searches, downloads, and organizes movies |
| **Lidarr** | 8686 | Music collection manager - searches, downloads, and organizes music |
| **Huntarr** | 9705 | Music discovery tool - finds and suggests new music |
| **Prowlarr** | 9696 | Indexer manager - centralized configuration for torrent/usenet sources |
| **qBittorrent** | 9865 | BitTorrent client with web interface |
| **FlareSolverr** | 8191 | CloudFlare solver for protected tracker access |

## Quick Start

### 1. Prerequisites
- Docker and Docker Compose installed
- Sufficient disk space for media and downloads

### 2. Configuration
1. Clone or download this repository
2. Copy the example environment file and customize it:
   ```bash
   cp .env.example .env
   ```
3. Edit `.env` to match your system:
   ```bash
   # Adjust these paths to match your system
   BASE_PATH=/your/docker/path
   MEDIA_PATH=/your/media/path
   
   # Set your user/group IDs (find with: id $(whoami))
   PUID=1000
   PGID=1000
   
   # Set your timezone
   TZ=America/New_York
   ```

### 3. Directory Structure
Create the required directories:
```bash
mkdir -p /your/docker/path/{radarr,lidarr,huntarr,prowlarr,qbittorrent}/{config}
mkdir -p /your/docker/path/qbittorrent/{downloads,incomplete}
mkdir -p /your/media/path/{video/Movie,music}
```

### 4. Deploy
```bash
docker-compose up -d
```

### 5. Initial Setup
1. **qBittorrent** (http://localhost:9865)
   - Default login: `admin` / `adminadmin`
   - Change password immediately
   - Configure download paths: `/downloads` and `/incomplete`

2. **Prowlarr** (http://localhost:9696)
   - Add indexers (torrent sites)
   - Connect to Radarr and Lidarr via Apps section

3. **Radarr** (http://localhost:7878)
   - Add download client (qBittorrent)
   - Configure media folder: `/movies`
   - Prowlarr will automatically sync indexers

4. **Lidarr** (http://localhost:8686)
   - Add download client (qBittorrent)
   - Configure media folder: `/music`
   - Prowlarr will automatically sync indexers

5. **Huntarr** (http://localhost:9705)
   - Connect to Lidarr for automated music discovery

## Environment Variables

### Core Settings
```bash
PUID=1000                    # User ID for file permissions
PGID=1000                    # Group ID for file permissions  
TZ=America/New_York          # Timezone
UMASK=022                    # File creation mask
```

### Paths
```bash
BASE_PATH=/volume1/docker              # Base path for container configs
MEDIA_PATH=/volume1                    # Base path for media storage
DOWNLOADS_PATH=/volume1/docker/qbittorrent/downloads
```

### Ports
```bash
RADARR_PORT=7878
LIDARR_PORT=8686
HUNTARR_PORT=9705
PROWLARR_PORT=9696
QBITTORRENT_PORT=9865
FLARESOLVERR_PORT=8191
```

### qBittorrent Settings
```bash
QBITTORRENT_MEMORY_LIMIT=2g
FILESYSTEM_CACHE=1G
WEBUI_AUTH_SUBNETWHITELIST=192.168.1.0/24,172.19.0.0/16
WEBUI_AUTH_LOCAL_AUTH_BYPASS=true
```

## File Permissions

The stack uses PUID and PGID to ensure proper file permissions. To find your user/group IDs:
```bash
id $(whoami)
```

## Security Notes

- Change qBittorrent default password immediately
- Adjust `WEBUI_AUTH_SUBNETWHITELIST` to match your network
- Consider using a reverse proxy (nginx, Traefik) for external access
- Keep containers updated regularly

## Troubleshooting

### Common Issues

**Permission Errors**
- Verify PUID/PGID match your system user
- Check directory ownership: `chown -R $PUID:$PGID /path/to/directories`

**Can't Access Web Interfaces**
- Check if ports are already in use: `netstat -tulpn | grep :PORT`
- Verify firewall settings
- Check container logs: `docker-compose logs SERVICE_NAME`

**Download Issues**
- Verify qBittorrent download paths
- Check indexer configuration in Prowlarr
- Ensure FlareSolverr is running for CloudFlare-protected sites

**Connection Issues Between Services**
- Services communicate using container names (e.g., `qbittorrent:9865`)
- Check network connectivity: `docker-compose exec SERVICE_NAME ping OTHER_SERVICE`

### Useful Commands

```bash
# View logs
docker-compose logs -f [service_name]

# Restart a service
docker-compose restart [service_name]

# Update containers
docker-compose pull
docker-compose up -d

# Check container status
docker-compose ps

# Access container shell
docker-compose exec [service_name] bash
```

## Backup

### Important Directories to Backup
- `${BASE_PATH}/*/config` - All application configurations
- Your media directories (movies, music)

### Backup Script Example
```bash
#!/bin/bash
BACKUP_DIR="/path/to/backups/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Backup configurations
tar -czf "$BACKUP_DIR/configs.tar.gz" -C /your/docker/path \
  radarr/config lidarr/config prowlarr/config qbittorrent/config huntarr/config

# Stop containers for consistent backup (optional)
docker-compose stop
# Backup, then restart
docker-compose start
```

## Updates

Keep your stack updated:
```bash
# Pull latest images
docker-compose pull

# Recreate containers with new images
docker-compose up -d

# Clean up old images
docker image prune
```

## Support

- **Radarr**: https://radarr.video/
- **Lidarr**: https://lidarr.audio/
- **Prowlarr**: https://prowlarr.com/
- **qBittorrent**: https://www.qbittorrent.org/
- **FlareSolverr**: https://github.com/FlareSolverr/FlareSolverr

## License

This configuration is provided as-is. Please ensure you comply with local laws regarding media downloading and sharing.