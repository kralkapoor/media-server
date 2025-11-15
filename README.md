# A Personal Media Server!

A containerised self-hosted media stack with automated content management and VPN-protected downloads.

## Architecture

This stack runs a complete media stack environment with the following components:

- **Gluetun**: VPN container using ProtonVPN with WireGuard
- **qBittorrent**: Download client routed through VPN
- **Prowlarr**: Indexer management for the automation tools
- **Radarr/Sonarr**: Automated movie and TV show management
- **Jellyfin**: Media server with hardware transcoding support
- **Jellyseerr**: Content request and approval interface
- **Nginx Proxy Manager**: Reverse proxy for external access
- **Tdarr**: Automated transcodes with optional HWA support

### Port Forwarding Automation

The stack includes a custom "port-forward sync" container that monitors the VPN's forwarded port and automatically updates qBittorrent when changes occur. This is helpful if your VPN provider rotates the forwarded port regularly (a la ProtonVPN).

This can be removed if you do not intend to portforward.

## Requirements

- Docker and Docker Compose
- VPN with WireGuard configuration and portforward support (e.g. ProtonVPN)
- Some variety of server (e.g. home server made of spare parts)
  - CPU: ideally an Intel chip with Quick Sync for better transcode performance, or whatever is laying around.
  - GPU: optional GPU for HWA
  - Storage: whatever you can scavenge!
- Domain name (optional, for SSL certificates)

## Installation

### 1. Environment Configuration

Create a `.env` file with the following variables:
```bash
# User and group IDs
PUID=1000
PGID=1000

# Timezone, e.g.
TZ=Pacific/Auckland

# Storage paths
DOWNLOADS_DIR=/path/to/downloads
MOVIES_DIR=/path/to/movies  
TV_DIR=/path/to/tv

# ProtonVPN WireGuard configuration
WIREGUARD_KEY=your_private_key
WIREGUARD_ADDRESSES=10.x.x.x/32
VPN_SERVER_COUNTRIES=Genovia,Wakanda  # Countries with port forwarding support, if using portforwarding.
```

### 2. Directory Structure

The services should create and mount the following directories. You can optionally create them yourself manually:
```bash
mkdir -p downloads media/{movies,tv}
mkdir -p gluetun qbittorrent/appdata 
mkdir -p nginx/{data,letsencrypt}
mkdir -p jellyfin/library jellyseerr/config
mkdir -p {radarr,sonarr,prowlarr}/data
```

### 3. Deployment

```bash
docker-compose up -d
```

### 4. Initial Setup

1. Configure indexers etc in the arr services.
2. Access Nginx Proxy Manager at `http://<server-ip>:81`
3. Configure proxy hosts for Jellyfin and Jellyseerr
4. Enable SSL certificates through Let's Encrypt if using a domain
5. Ensure firewall rules are updated, e.g. in `ufw`. It is recommended to only allow access to non-internet exposed services from within your local network.

## Network Configuration

External access is handled exclusively through Nginx Proxy Manager on ports 80 and 443. All other services operate on an internal Docker network and are not directly exposed.

The download workflow follows this path:
- User request → Jellyseerr
- Approval → Radarr/Sonarr
- Indexer search → Prowlarr
- Download → qBittorrent (via Gluetun network)

## Service Monitoring

### VPN Status

Verify VPN connection:
```bash
docker exec gluetun curl http://127.0.0.1:8000/v1/vpn/status
```

Check forwarded port:
```bash
docker exec gluetun curl http://127.0.0.1:8000/v1/portforward
```

## Technical Notes

- Hardware transcoding is enabled for Jellyfin via `/dev/dri` device pass-through
- All containers include health checks and will restart automatically on failure
- Download traffic is isolated to the Gluetun network. If Gluetun or your VPN drops, qBitTorrent will be unreachable.
- The port sync container checks for changes every 60 seconds. TODO should be a configurable env variable

## Troubleshooting

### qBittorrent Connection Issues
Allow time for the VPN to establish connection and port forwarding to complete. Check the portforward-sync logs to confirm the port has been updated.

### External Access Problems
Verify Nginx Proxy Manager configuration and ensure ports 80 and 443 are accessible from outside your network.
Check your domain records if applicable and make sure they are pointing to the right IP address. This assumes you have a static IP address.

### Jellyfin Transcoding
Ensure the Docker user has appropriate permissions for `/dev/dri` (typically requires membership in the `render` or `video` group).
