# Media stack

The most fun you can have with spare hardware!

Currently configured using Gluetun and ProtonVPN with Wireguard

## Exposed services

| Service    | Port |
|------------|------|
| Jellyfin   | 8096 |
| Jellyseerr | 5055 |

## Internal services

| Service     | Port |
|-------------|------|
| qBittorrent | 8080 |
| Prowlarr    | 9696 |
| Radarr      | 7878 |
| Sonarr      | 8989 |

## Environment vars

Override as required

```
PUID=1000
PGID=1000
TZ=Pacific/Auckland
DOWNLOADS_DIR=path/to/downloads/dir
MOVIES_DIR=path/to/movies/dir
TV_DIR=path/to/tv/dir
WIREGUARD_KEY=key
WIREGUARD_ADDRESSES=0.0.0.0/0
VPN_SERVER_COUNTRIES=country1,country2,country3
```
