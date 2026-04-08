<div align="center">

# Fitz Homelab

**Self-hosted everything on a Ugreen DXP4800 Plus**

Media automation, photo management, document archival, password vault, cloud storage, and network-wide ad blocking - all running on a single NAS.

[![Docker](https://img.shields.io/badge/Docker-23_Containers-2496ED?style=for-the-badge&logo=docker&logoColor=white)](docker-compose/)
[![Compose](https://img.shields.io/badge/Compose-4_Stacks-2496ED?style=for-the-badge&logo=docker&logoColor=white)](#docker-compose-stacks)
[![Tailscale](https://img.shields.io/badge/Tailscale-Mesh_VPN-4C8BF5?style=for-the-badge&logo=tailscale&logoColor=white)](https://tailscale.com/)
[![Ugreen](https://img.shields.io/badge/Ugreen-DXP4800_Plus-4CAF50?style=for-the-badge&logoColor=white)](#hardware)

</div>

---

## The Stack

### Media Automation

> The arr suite - fully automated media acquisition, organization, and subtitle management.

[![Sonarr](https://img.shields.io/badge/Sonarr-:8989-2196F3?style=flat-square&logo=sonarr&logoColor=white)](https://sonarr.tv/)
[![Radarr](https://img.shields.io/badge/Radarr-:7878-FFC107?style=flat-square&logo=radarr&logoColor=black)](https://radarr.video/)
[![Prowlarr](https://img.shields.io/badge/Prowlarr-:9696-E040FB?style=flat-square&logo=prowlarr&logoColor=white)](https://prowlarr.com/)
[![Bazarr](https://img.shields.io/badge/Bazarr-:6767-9C27B0?style=flat-square&logo=bazarr&logoColor=white)](https://www.bazarr.media/)
[![Recyclarr](https://img.shields.io/badge/Recyclarr-Daily_Sync-00BCD4?style=flat-square)](https://recyclarr.dev/)
[![FlareSolverr](https://img.shields.io/badge/FlareSolverr-:8191-FF5722?style=flat-square)](https://github.com/FlareSolverr/FlareSolverr)

### Download Clients

[![SABnzbd](https://img.shields.io/badge/SABnzbd-:8085-F9A825?style=flat-square&logo=sabnzbd&logoColor=black)](https://sabnzbd.org/)
[![qBittorrent](https://img.shields.io/badge/qBittorrent-VPN-2F67BA?style=flat-square&logo=qbittorrent&logoColor=white)](https://www.qbittorrent.org/)

### Media Playback

> Jellyfin with hardware transcoding, request management via Jellyseerr, and Discord integration.

[![Jellyfin](https://img.shields.io/badge/Jellyfin-:8899-A359D6?style=flat-square&logo=jellyfin&logoColor=white)](https://jellyfin.org/)
[![Jellyseerr](https://img.shields.io/badge/Jellyseerr-:5055-7B2FBE?style=flat-square)](https://github.com/Fallenbagel/jellyseerr)
[![Requestrr](https://img.shields.io/badge/Requestrr-:4545-5865F2?style=flat-square&logo=discord&logoColor=white)](https://github.com/thomst08/requestrr)

### Photos & Documents

> Immich for photos with ML-powered face/object detection. Paperless-ngx for document OCR and archival.

[![Immich](https://img.shields.io/badge/Immich-:2283-4250AF?style=flat-square&logo=immich&logoColor=white)](https://immich.app/)
[![Paperless-ngx](https://img.shields.io/badge/Paperless--ngx-:8010-17541f?style=flat-square)](https://docs.paperless-ngx.com/)

### Cloud & Files

[![Nextcloud](https://img.shields.io/badge/Nextcloud-:35041-0082C9?style=flat-square&logo=nextcloud&logoColor=white)](https://nextcloud.com/)

### Security & Access

> Vaultwarden with zero exposed ports - only accessible via Tailscale Serve HTTPS. AdGuard as network DNS.

[![Vaultwarden](https://img.shields.io/badge/Vaultwarden-Tailscale_Only-175DDC?style=flat-square&logo=bitwarden&logoColor=white)](https://github.com/dani-garcia/vaultwarden)
[![AdGuard Home](https://img.shields.io/badge/AdGuard_Home-macvlan-68BC71?style=flat-square&logo=adguard&logoColor=white)](https://adguard.com/en/adguard-home/overview.html)
[![Tailscale](https://img.shields.io/badge/Tailscale-Mesh_VPN-4C8BF5?style=flat-square&logo=tailscale&logoColor=white)](https://tailscale.com/)

### System & Management

[![Portainer](https://img.shields.io/badge/Portainer-:9444-13BEF9?style=flat-square&logo=portainer&logoColor=white)](https://www.portainer.io/)
[![Duplicati](https://img.shields.io/badge/Duplicati-:8200_AES--256-2E7D32?style=flat-square)](https://www.duplicati.com/)

---

## Hardware

| | Component | Details |
|---|-----------|---------|
| **NAS** | Ugreen DXP4800 Plus | Intel QuickSync HW transcoding |
| **Storage** | Multi-volume | Configs on Vol 2, Media on Vol 1 |
| **Client** | NVIDIA Shield TV Pro (2019) | HDR10, HDR10+, Dolby Vision |
| **Audio** | LG SN11RG | 7.1.4 Dolby Atmos, TrueHD passthrough |
| **VPN** | Tailscale | Mesh network, zero open ports |
| **DNS** | AdGuard Home | macvlan, dedicated LAN IP |

---

## Architecture

```mermaid
graph TB
    Internet((Internet))
    Router[Router / Gateway]
    TS[Tailscale Mesh VPN]

    Internet --> Router
    Router --> TS
    Router --> NAS

    subgraph NAS["Ugreen DXP4800 Plus"]
        direction TB

        subgraph arr["arr-stack  |  12 containers"]
            direction LR
            Prowlarr --> Sonarr
            Prowlarr --> Radarr
            Sonarr --> SABnzbd
            Radarr --> SABnzbd
            Sonarr --> qBit[qBittorrent]
            Radarr --> qBit
            SABnzbd --> Jellyfin
            qBit --> Jellyfin
            Jellyseerr --> Sonarr
            Jellyseerr --> Radarr
            Jellyseerr --> Jellyfin
            Bazarr --> Sonarr
            Bazarr --> Radarr
        end

        subgraph photos["photos & docs"]
            direction LR
            ImmichServer[Immich] --> ImmichML[Immich ML]
            ImmichServer --> ImmichPG[(PostgreSQL)]
            ImmichServer --> ImmichRedis[(Valkey)]
            Paperless[Paperless-ngx] --> PaperlessPG[(PostgreSQL)]
            Paperless --> PaperlessRedis[(Redis)]
        end

        subgraph vault["vaultwarden"]
            direction LR
            TSProxy[Tailscale Sidecar] --> VW[Vaultwarden]
        end

        subgraph standalone["standalone"]
            direction LR
            Nextcloud --> NCPG[(PostgreSQL)]
            Portainer
        end

        subgraph dns["macvlan network"]
            AdGuard[AdGuard Home]
        end

        Duplicati
        Recyclarr
        Requestrr
    end

    TS -->|HTTPS| TSProxy

    style NAS fill:#1a1a2e,stroke:#4CAF50,stroke-width:2px,color:#fff
    style arr fill:#1e3a5f,stroke:#2196F3,stroke-width:1px,color:#fff
    style photos fill:#1e3a5f,stroke:#4250AF,stroke-width:1px,color:#fff
    style vault fill:#1e3a5f,stroke:#175DDC,stroke-width:1px,color:#fff
    style standalone fill:#1e3a5f,stroke:#0082C9,stroke-width:1px,color:#fff
    style dns fill:#1e3a5f,stroke:#68BC71,stroke-width:1px,color:#fff
    style TS fill:#4C8BF5,stroke:#4C8BF5,color:#fff
    style Internet fill:#333,stroke:#666,color:#fff
    style Router fill:#333,stroke:#666,color:#fff
```

### Media Flow

```mermaid
graph LR
    A["Request via Jellyseerr\nor Discord bot"] --> B[Sonarr / Radarr]
    B --> C[Prowlarr\nindexer search]
    C --> D["SABnzbd / qBittorrent\ndownload"]
    D --> E[Sonarr / Radarr\nimport & rename]
    E --> F["Jellyfin\nstream to Shield TV"]
    E --> G[Bazarr\nfetch subtitles]

    style A fill:#7B2FBE,stroke:#7B2FBE,color:#fff
    style B fill:#2196F3,stroke:#2196F3,color:#fff
    style C fill:#E040FB,stroke:#E040FB,color:#fff
    style D fill:#F9A825,stroke:#F9A825,color:#000
    style E fill:#2196F3,stroke:#2196F3,color:#fff
    style F fill:#A359D6,stroke:#A359D6,color:#fff
    style G fill:#9C27B0,stroke:#9C27B0,color:#fff
```

### Vaultwarden via Tailscale Serve

Zero ports exposed to LAN or internet. The Tailscale sidecar handles all networking:

```mermaid
graph LR
    Device["Any Tailscale Device"] -->|HTTPS| TSNet["vaultwarden.tailnet.ts.net"]
    TSNet --> TSC["ts-vaultwarden container\n(Tailscale Serve)"]
    TSC -->|"network_mode: service"| VW["vaultwarden container"]

    style Device fill:#333,stroke:#666,color:#fff
    style TSNet fill:#4C8BF5,stroke:#4C8BF5,color:#fff
    style TSC fill:#4C8BF5,stroke:#4C8BF5,color:#fff
    style VW fill:#175DDC,stroke:#175DDC,color:#fff
```

---

## Recyclarr / TRaSH Guides

Quality profiles synced daily, optimized for the home theater setup:

| Target | Optimization |
|--------|-------------|
| **NVIDIA Shield TV Pro** | HDR10, HDR10+, Dolby Vision (Profile 5/7/8) |
| **LG SN11RG Soundbar** | TrueHD Atmos, DTS-X, DTS-HD MA passthrough |
| **Quality Priority** | Remux > WEB-DL with automatic upgrade paths |
| **Audio Priority** | Lossless first (TrueHD Atmos > DTS-HD MA > DD+ Atmos) |

Custom format definitions: [`recyclarr/custom-formats/`](recyclarr/custom-formats/)

---

## Docker Compose Stacks

| Stack | Containers | Config |
|-------|:----------:|--------|
| **arr-stack** | 12 | [`arr-stack.yml`](docker-compose/arr-stack.yml) |
| **jellyfin-stack** | 1 | Standalone Jellyfin instance |
| **paperless** | 3 | [`paperless.yml`](docker-compose/paperless.yml) |
| **vaultwarden** | 2 | [`vaultwarden.yml`](docker-compose/vaultwarden.yml) |
| **standalone** | ~5 | Nextcloud, Portainer, PostgreSQL |

<details>
<summary><strong>Volume Layout</strong></summary>

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

</details>

<details>
<summary><strong>Getting Started</strong></summary>

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

</details>

---

## Security

| | Measure | Details |
|---|---------|---------|
| ![shield](https://img.shields.io/badge/-PASS-2E7D32?style=flat-square) | **Zero exposed ports** | All remote access via Tailscale mesh VPN |
| ![shield](https://img.shields.io/badge/-PASS-2E7D32?style=flat-square) | **Vaultwarden isolated** | Tailscale Serve sidecar - not even on LAN |
| ![shield](https://img.shields.io/badge/-PASS-2E7D32?style=flat-square) | **DNS filtering** | AdGuard Home on macvlan with dedicated IP |
| ![shield](https://img.shields.io/badge/-PASS-2E7D32?style=flat-square) | **Encrypted backups** | Duplicati with AES-256 encryption |
| ![shield](https://img.shields.io/badge/-PASS-2E7D32?style=flat-square) | **Secrets management** | `.env` files with `chmod 600` permissions |
| ![shield](https://img.shields.io/badge/-PASS-2E7D32?style=flat-square) | **No secrets in repo** | `.gitignore` blocks all sensitive files |

> **If you fork this:** The `.gitignore` blocks all `.env` files and sensitive configs, but always double-check `git status` before pushing.

---

<div align="center">

### Built With

[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![LinuxServer](https://img.shields.io/badge/LinuxServer.io-DA3B8A?style=for-the-badge&logo=linux&logoColor=white)](https://www.linuxserver.io/)
[![TRaSH Guides](https://img.shields.io/badge/TRaSH_Guides-FFA500?style=for-the-badge)](https://trash-guides.info/)
[![Tailscale](https://img.shields.io/badge/Tailscale-4C8BF5?style=for-the-badge&logo=tailscale&logoColor=white)](https://tailscale.com/)

---

[![r/selfhosted](https://img.shields.io/badge/r%2Fselfhosted-FF4500?style=flat-square&logo=reddit&logoColor=white)](https://www.reddit.com/r/selfhosted/)
[![r/homelab](https://img.shields.io/badge/r%2Fhomelab-FF4500?style=flat-square&logo=reddit&logoColor=white)](https://www.reddit.com/r/homelab/)

*Running on a Ugreen DXP4800 Plus and an unhealthy amount of tinkering.*

</div>
