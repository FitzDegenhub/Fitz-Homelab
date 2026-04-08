# Fitz Homelab

Self-hosted media automation, document management, password vault, and more - running on a **Ugreen DXP4800 Plus** with Docker.

---

## Hardware

| Component | Details |
|-----------|---------|
| **NAS** | Ugreen DXP4800 Plus |
| **Transcoding** | Intel QuickSync (hardware via `/dev/dri`) |
| **Storage** | Multi-volume - configs on Volume 2, media on Volume 1 |
| **Clients** | NVIDIA Shield TV Pro (2019) |
| **Audio** | LG SN11RG 7.1.4 Dolby Atmos soundbar |
| **Remote Access** | Tailscale mesh VPN |
| **DNS** | AdGuard Home (macvlan, dedicated LAN IP) |

## Services

### Media Automation (arr-stack)

| Service | Port | Purpose |
|---------|------|---------|
| [Sonarr](https://sonarr.tv/) | `8989` | TV show management & automation |
| [Radarr](https://radarr.video/) | `7878` | Movie management & automation |
| [Prowlarr](https://prowlarr.com/) | `9696` | Indexer management for Sonarr/Radarr |
| [Bazarr](https://www.bazarr.media/) | `6767` | Automated subtitle downloads |
| [Recyclarr](https://recyclarr.dev/) | - | TRaSH Guides quality profile sync (daily) |
| [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr) | `8191` | Cloudflare bypass for indexers |

### Download Clients

| Service | Port | Purpose |
|---------|------|---------|
| [SABnzbd](https://sabnzbd.org/) | `8085` | Usenet downloader |
| [qBittorrent](https://www.qbittorrent.org/) | - | Torrent client (linuxserver) |

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
| Immich ML | - | Face/object detection & smart search |
| PostgreSQL (vectorchord) + Valkey | - | Immich backend services |

### Documents

| Service | Port | Purpose |
|---------|------|---------|
| [Paperless-ngx](https://docs.paperless-ngx.com/) | `8010` | Document scanner, OCR, and archive |
| PostgreSQL 16 + Redis 7 | - | Paperless backend services |

### Cloud & Files

| Service | Port | Purpose |
|---------|------|---------|
| [Nextcloud](https://nextcloud.com/) | `35041` | Self-hosted file sync & cloud storage |
| PostgreSQL | - | Nextcloud database |

### Security & Access

| Service | Port | Purpose |
|---------|------|---------|
| [Vaultwarden](https://github.com/dani-garcia/vaultwarden) | - | Self-hosted Bitwarden password manager |
| [Tailscale](https://tailscale.com/) sidecar | - | Vaultwarden accessible only via Tailscale Serve (HTTPS) |
| [AdGuard Home](https://adguard.com/en/adguard-home/overview.html) | macvlan | Network-wide DNS ad blocking |

### System & Management

| Service | Port | Purpose |
|---------|------|---------|
| [Portainer CE](https://www.portainer.io/) | `9444` | Docker management UI |
| [Duplicati](https://www.duplicati.com/) | `8200` | Encrypted backups (AES-256) |

## Architecture

```
Internet
    |
[Router/Gateway]
    |
    +--- Tailscale Mesh VPN
    |       +--- Vaultwarden (HTTPS via Tailscale Serve, zero exposed ports)
    |       +--- All services accessible remotely
    |
[Ugreen DXP4800 Plus]
    |
    +--- arr-stack (Docker Compose - 12 containers)
    |       +--- Prowlarr -> Sonarr/Radarr
    |       +--- Sonarr/Radarr -> SABnzbd/qBittorrent
    |       +--- Jellyfin (HW transcoding via Intel QuickSync)
    |       +--- Jellyseerr -> Jellyfin + Sonarr/Radarr
    |       +--- Immich + ML + PostgreSQL + Valkey
    |       +--- Bazarr, Recyclarr, FlareSolverr
    |       +--- Duplicati, Requestrr
    |
    +--- jellyfin-stack (Docker Compose - 1 container)
    |       +--- Jellyfin (standalone instance)
    |
    +--- paperless (Docker Compose - 3 containers)
    |       +--- Paperless-ngx + PostgreSQL 16 + Redis 7
    |
    +--- vaultwarden (Docker Compose - 2 containers)
    |       +--- Vaultwarden + Tailscale sidecar
    |
    +--- Standalone containers
    |       +--- Nextcloud + PostgreSQL
    |       +--- Portainer CE
    |
    +--- macvlan Network
            +--- AdGuard Home (dedicated LAN IP as network DNS)
```

**Total: ~23 containers across 4 Compose stacks + standalone services**

### Media Flow

```
Request (Jellyseerr / Requestrr via Discord)
    -> Sonarr / Radarr (search & organize)
    -> Prowlarr (indexer queries)
    -> SABnzbd / qBittorrent (download)
    -> Sonarr / Radarr (import & rename)
    -> Jellyfin (stream to Shield TV Pro)
    -> Bazarr (fetch subtitles)
```

### Vaultwarden via Tailscale Serve

The Vaultwarden setup uses a **Tailscale sidecar pattern** - zero ports are exposed to the LAN or internet. The Tailscale container handles networking and serves Vaultwarden over HTTPS on the tailnet:

```
[Any Tailscale device] --HTTPS--> vaultwarden.tailnet.ts.net
    -> ts-vaultwarden container (Tailscale Serve reverse proxy)
    -> vaultwarden container (network_mode: service:ts-vaultwarden)
```

### Recyclarr / TRaSH Guides

Quality profiles are synced daily via Recyclarr, optimized for:
- **NVIDIA Shield TV Pro** (HDR10, HDR10+, Dolby Vision Profile 5/7/8)
- **Dolby Atmos passthrough** via LG SN11RG soundbar
- Lossless audio priority (TrueHD Atmos, DTS-HD MA)
- Tiered quality: Remux > WEB-DL with upgrade paths

Custom format definitions are in [`recyclarr/custom-formats/`](recyclarr/custom-formats/).

## Docker Compose Stacks

| Stack | Containers | Compose File |
|-------|-----------|--------------|
| **arr-stack** | 12 | [`docker-compose/arr-stack.yml`](docker-compose/arr-stack.yml) |
| **jellyfin-stack** | 1 | Standalone Jellyfin instance |
| **paperless** | 3 | [`docker-compose/paperless.yml`](docker-compose/paperless.yml) |
| **vaultwarden** | 2 | [`docker-compose/vaultwarden.yml`](docker-compose/vaultwarden.yml) |
| **standalone** | ~5 | Nextcloud, Portainer, PostgreSQL (managed via Portainer) |

## Volume Layout

```
/volume1/Media/
    Media/
        Movies/
        TV Shows/
        Anime/
        Music/
    Photos/              # Immich library
    Torrents/            # Active downloads
    usenet/              # Usenet downloads
        complete/
        incomplete/
    Backups/             # Duplicati targets

/volume2/docker/
    arr-stack/           # Media automation configs
        sonarr/config/
        radarr/config/
        prowlarr/config/
        jellyfin/config/
        immich/
            postgres/
            model-cache/
        dispatcharr/
        ...
    jellyfin-stack/      # Standalone Jellyfin
    paperless/           # Document management
        data/
        media/
        consume/         # Drop PDFs here for auto-import
        postgres/
    vaultwarden/         # Password manager
        data/
        tailscale/
            state/
            config/      # serve.json for TS Serve
    nextcloud/
    postgres-1/
```

## Getting Started

### Prerequisites

- Docker & Docker Compose on your NAS
- Tailscale account (for remote access & Vaultwarden)

### Setup

1. Clone this repo:
   ```bash
   git clone https://github.com/FitzDegenhub/Fitz-Homelab.git
   cd Fitz-Homelab
   ```

2. Copy and configure your environment:
   ```bash
   cp .env.example .env
   # Edit .env with your actual values
   ```

3. Deploy individual stacks:
   ```bash
   # Media automation
   docker compose -f docker-compose/arr-stack.yml --env-file .env up -d

   # Document management
   docker compose -f docker-compose/paperless.yml --env-file .env up -d

   # Password manager (configure Tailscale auth key first)
   docker compose -f docker-compose/vaultwarden.yml up -d
   ```

## Security

- **Zero ports exposed to the internet** - all remote access via Tailscale
- **Vaultwarden** uses Tailscale Serve sidecar - not even exposed on LAN
- **AdGuard Home** on macvlan with dedicated IP as network DNS
- **Duplicati** backups encrypted with AES-256
- Secrets managed via `.env` files with `chmod 600` permissions
- API keys and passwords are never committed to this repo

> **If you fork this:** The `.gitignore` blocks all `.env` files and sensitive configs, but always double-check `git status` before pushing.

## Acknowledgements

- [TRaSH Guides](https://trash-guides.info/) - Quality profile configurations
- [LinuxServer.io](https://www.linuxserver.io/) - Docker images
- [r/selfhosted](https://www.reddit.com/r/selfhosted/) - Inspiration and troubleshooting
- [r/homelab](https://www.reddit.com/r/homelab/) - The rabbit hole that started it all

---

*Running on a Ugreen DXP4800 Plus and an unhealthy amount of tinkering.*
