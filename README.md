# My Homelab Stack

Self-hosted media automation, photo management, and network services running on a **Ugreen NAS** with Docker.

<!-- TODO: Add a screenshot of your Homepage dashboard here -->
<!-- ![Dashboard](docs/screenshots/dashboard.png) -->

---

## Hardware

| Component | Details |
|-----------|---------|
| **NAS** | Ugreen DXP series |
| **Transcoding** | Intel QuickSync (hardware via `/dev/dri`) |
| **Storage** | Multiple volumes - config on Volume 2, media on Volume 1 |
| **Clients** | NVIDIA Shield TV Pro (2019), LG CX OLED |
| **Audio** | LG SN11RG 7.1.4 Dolby Atmos soundbar |
| **Remote Access** | Tailscale VPN |
| **DNS** | AdGuard Home (macvlan, dedicated IP) |

## Services

### Media Automation

| Service | Port | Purpose |
|---------|------|---------|
| [Sonarr](https://sonarr.tv/) | `8989` | TV show management & automation |
| [Radarr](https://radarr.video/) | `7878` | Movie management & automation |
| [Prowlarr](https://prowlarr.com/) | `9696` | Indexer management for Sonarr/Radarr |
| [Bazarr](https://www.bazarr.media/) | `6767` | Automated subtitle downloads |
| [Recyclarr](https://recyclarr.dev/) | - | TRaSH Guides quality profile sync |
| [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr) | `8191` | Cloudflare bypass for indexers |

### Download Clients

| Service | Port | Purpose |
|---------|------|---------|
| [SABnzbd](https://sabnzbd.org/) | `8085` | Usenet downloader |
| qBittorrent | `8888` | Torrent client |

### Media Playback

| Service | Port | Purpose |
|---------|------|---------|
| [Jellyfin](https://jellyfin.org/) | `8899` | Media server with HW transcoding |
| [Jellyseerr](https://github.com/Fallenbagel/jellyseerr) | `5055` | Media request management |
| [Requestrr](https://github.com/thomst08/requestrr) | `4545` | Discord bot for media requests |

### Photos

| Service | Port | Purpose |
|---------|------|---------|
| [Immich](https://immich.app/) | `2283` | Self-hosted Google Photos alternative |
| Immich ML | - | Machine learning for face/object detection |
| PostgreSQL + Valkey | - | Immich backend services |

### Network & System

| Service | Port | Purpose |
|---------|------|---------|
| [AdGuard Home](https://adguard.com/en/adguard-home/overview.html) | `3000` | Network-wide DNS ad blocking |
| [Homepage](https://gethomepage.dev/) | `3000` | Service dashboard |
| [Duplicati](https://www.duplicati.com/) | `8200` | Encrypted backups (AES-256) |
| [Tailscale](https://tailscale.com/) | - | Mesh VPN for secure remote access |

## Architecture

```
Internet
    |
[Router/Gateway]
    |
    +--- Tailscale VPN (remote access)
    |
[Ugreen NAS - 192.168.x.x]
    |
    +--- Docker Bridge Network (arr-net)
    |       +--- Prowlarr -> Sonarr/Radarr
    |       +--- Sonarr/Radarr -> SABnzbd/qBittorrent
    |       +--- Jellyfin (HW transcoding)
    |       +--- Jellyseerr -> Jellyfin + Sonarr/Radarr
    |       +--- Immich + ML + PostgreSQL + Valkey
    |       +--- Homepage (dashboard)
    |       +--- Duplicati (encrypted backups)
    |
    +--- macvlan Network (adguard-net)
            +--- AdGuard Home (dedicated IP as network DNS)
```

### Media Flow

```
Request (Jellyseerr/Requestrr)
    -> Sonarr/Radarr (search & organize)
    -> Prowlarr (indexer queries)
    -> SABnzbd/qBittorrent (download)
    -> Sonarr/Radarr (import & rename)
    -> Jellyfin (stream to devices)
    -> Bazarr (fetch subtitles)
```

### Recyclarr / TRaSH Guides

Quality profiles are synced daily via Recyclarr, optimized for:
- **NVIDIA Shield TV Pro** (HDR10, HDR10+, Dolby Vision Profile 5/7/8)
- **Dolby Atmos passthrough** via LG SN11RG soundbar
- Lossless audio priority (TrueHD Atmos, DTS-HD MA)
- Tiered quality: Remux > WEB-DL with upgrade paths

## Volume Layout

```
/volume1/Media/
    Media/
        Movies/
        TV Shows/
        Anime/
        Music/
    Photos/          # Immich library
    Torrents/        # Active downloads
    usenet/          # Usenet downloads
        complete/
        incomplete/
    Backups/         # Duplicati targets

/volume2/docker/
    arr-stack/       # All container configs
        sonarr/config/
        radarr/config/
        prowlarr/config/
        jellyfin/config/
        immich/
            postgres/
            model-cache/
        homepage/config/
        ...
```

## Getting Started

### Prerequisites

- Docker & Docker Compose installed on your NAS
- Tailscale account (for remote access)

### Setup

1. Clone this repo:
   ```bash
   git clone https://github.com/YOUR_USERNAME/homelab.git
   cd homelab
   ```

2. Copy and configure your environment:
   ```bash
   cp .env.example .env
   # Edit .env with your actual values (IPs, API keys, passwords)
   ```

3. Copy Homepage configs:
   ```bash
   cp homepage/*.yaml.example /path/to/homepage/config/
   # Rename to remove .example suffix and fill in your values
   ```

4. Deploy the stack:
   ```bash
   docker compose -f docker-compose/arr-stack.yml --env-file .env up -d
   ```

5. Access your services via `http://your-nas-ip:PORT`

## Security Notes

- All remote access goes through **Tailscale** - no ports exposed to the internet
- Secrets are managed via `.env` files with restricted permissions (`chmod 600`)
- AdGuard Home runs on a **macvlan** network with its own dedicated IP
- Duplicati backups use **AES-256 encryption**
- API keys and passwords are never committed to this repo

> **If you fork this:** Make sure you never commit your `.env` file. The `.gitignore` is configured to prevent this, but always double-check before pushing.

## Acknowledgements

- [TRaSH Guides](https://trash-guides.info/) - Quality profile configurations
- [LinuxServer.io](https://www.linuxserver.io/) - Docker images
- [Selfhosted community](https://www.reddit.com/r/selfhosted/) - Inspiration and troubleshooting

---

*Built with a Ugreen NAS and way too many hours of tinkering.*
