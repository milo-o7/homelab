# Software Setup Guide

Step-by-step configuration for all Docker containers on the M920q (Unraid) and Pi 5.

---

## Table of Contents

- [Unraid Initial Setup](#unraid-initial-setup)
- [Storage Configuration](#storage-configuration)
- [Jellyfin](#jellyfin)
- [Arr Stack (Sonarr, Radarr, Prowlarr)](#arr-stack)
- [qBittorrent + VPN](#qbittorrent--vpn)
- [Bazarr](#bazarr)
- [Immich](#immich)
- [Pi 5: AdGuard Home](#pi-5-adguard-home)
- [Pi 5: WireGuard](#pi-5-wireguard)
- [Pi 5: Uptime Kuma](#pi-5-uptime-kuma)

---

## Unraid Initial Setup

### Flash the USB Drive

1. Download the [Unraid USB Creator](https://unraid.net/download) on a Windows/Mac/Linux machine
2. Insert the SanDisk 16GB USB drive
3. Select "Stable" release, choose the USB drive, click Write
4. Boot the M920q from the USB drive

### First Boot

1. Connect the M920q to the switch via ethernet
2. Access the Unraid web GUI at `http://tower.local` or `http://<IP>:80`
3. Register your Starter license ($49) — enter your flash GUID
4. Set a root password under **Users > root**

### Install Community Applications

1. Go to **Plugins > Install Plugin**
2. Paste the Community Applications URL:
   ```
   https://raw.githubusercontent.com/Squidly271/community.applications/master/plugins/community.applications.plg
   ```
3. Click Install — this adds the "Apps" tab for one-click Docker container installation

---

## Storage Configuration

### Array Setup

1. Go to **Main > Array Devices**
2. Assign your first WD Red Plus as **Disk 1** (data)
3. Assign your second WD Red Plus as **Parity** (parity protection)
4. Assign your NVMe SSD as **Cache** (Docker containers + app data)
5. Click **Start** — parity sync will begin (~8-12 hours for 4TB)

### Create Shares

Set up the unified folder structure that enables hardlinks:

1. Go to **Shares > Add Share**
2. Create share named `data` with these settings:
   - Primary storage: Array
   - Cache: Prefer (writes to cache first, mover transfers to array)

3. SSH into the server and create the folder structure:
   ```bash
   mkdir -p /mnt/user/data/torrents/movies
   mkdir -p /mnt/user/data/torrents/tv
   mkdir -p /mnt/user/data/media/movies
   mkdir -p /mnt/user/data/media/tv
   mkdir -p /mnt/user/data/media/music
   ```

> This unified `/data/` structure is critical. All apps must see the same root path for hardlinks to work. Without this, Sonarr/Radarr will *copy* files instead of instant-linking them, doubling your disk usage.

---

## Jellyfin

**Purpose:** Media streaming server (replaces Netflix/Plex)

### Install

1. Go to **Apps** tab, search "Jellyfin"
2. Install the official `jellyfin/jellyfin` container
3. Key settings:
   - **Media path:** `/mnt/user/data/media` mapped to `/media` in container
   - **Config path:** `/mnt/user/appdata/jellyfin`
   - **Network:** Bridge mode, port `8096`
   - **Hardware transcoding:** Enable Intel QSV (UHD 630)

### Hardware Transcoding Setup

1. In Jellyfin: **Dashboard > Playback > Transcoding**
2. Hardware acceleration: **Intel QuickSync (QSV)**
3. Enable: HEVC decoding, HEVC 10-bit decoding, HDR tone mapping
4. In Unraid Docker settings, pass through `/dev/dri` for GPU access

### Access

- Local: `http://192.168.1.50:8096`
- Add libraries pointing to `/media/movies` and `/media/tv`

---

## Arr Stack

### Prowlarr (Indexer Manager)

**Install first** — Sonarr and Radarr depend on it.

1. **Apps** > search "Prowlarr" > install `linuxserver/prowlarr`
2. Port: `9696`
3. Config path: `/mnt/user/appdata/prowlarr`
4. Add indexers (torrent sites) through the web UI
5. Note the API key from **Settings > General** — Sonarr/Radarr will need it

### Sonarr (TV Shows)

1. **Apps** > search "Sonarr" > install `linuxserver/sonarr`
2. Port: `8989`
3. Path mappings:
   - `/mnt/user/data` -> `/data` (container sees everything under `/data`)
4. Configure:
   - **Settings > Indexers** > Add Prowlarr (paste API key, `http://prowlarr:9696`)
   - **Settings > Download Clients** > Add qBittorrent (`http://qbittorrent:8080`)
   - **Settings > Media Management** > Root folder: `/data/media/tv`

### Radarr (Movies)

1. **Apps** > search "Radarr" > install `linuxserver/radarr`
2. Port: `7878`
3. Same path mapping as Sonarr: `/mnt/user/data` -> `/data`
4. Configure:
   - Indexers: Prowlarr
   - Download client: qBittorrent
   - Root folder: `/data/media/movies`

---

## qBittorrent + VPN

Uses the `binhex-qbittorrentvpn` container — bundles qBittorrent + WireGuard + kill switch.

### Install

1. **Apps** > search "binhex-qbittorrentvpn" > install
2. Key settings:
   - Port: `8080` (web UI)
   - VPN_ENABLED: `yes`
   - VPN_PROV: `custom`
   - VPN_CLIENT: `wireguard`
   - Download path: `/data/torrents` (mapped to `/mnt/user/data/torrents`)

### Mullvad WireGuard Setup

1. Log into [mullvad.net](https://mullvad.net) > WireGuard configuration
2. Generate a config file
3. Place the `.conf` file in `/mnt/user/appdata/binhex-qbittorrentvpn/wireguard/`
4. Restart the container
5. Verify: In qBittorrent web UI, check your external IP — should show Mullvad server, not your real IP

### Kill Switch

The container has a built-in kill switch. If the VPN drops, all torrent traffic stops immediately. Your real IP is never exposed.

---

## Bazarr

**Purpose:** Automatic subtitle downloads for all media

1. **Apps** > search "Bazarr" > install `linuxserver/bazarr`
2. Port: `6767`
3. Path mapping: `/mnt/user/data/media` -> `/media`
4. Configure:
   - Connect to Sonarr and Radarr (API keys)
   - Add subtitle providers (OpenSubtitles.com account recommended)
   - Set preferred languages

---

## Immich

**Purpose:** Self-hosted Google Photos replacement

1. **Apps** > search "Immich" > install (follow the Unraid community template)
2. Port: `2283`
3. Photo storage: `/mnt/user/data/media/photos` (or a separate share)
4. Install Immich mobile app on your phone
5. Configure auto-backup of camera roll

> Immich is more complex than other containers — it requires PostgreSQL and Redis. The Unraid community template handles all dependencies automatically.

---

## Pi 5: AdGuard Home

### Install Docker + Portainer on Pi OS

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
# Log out and back in

# Install Portainer
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

### Install AdGuard Home

```bash
docker run -d --name adguardhome \
  --restart=always \
  -v /opt/adguardhome/work:/opt/adguardhome/work \
  -v /opt/adguardhome/conf:/opt/adguardhome/conf \
  -p 53:53/tcp -p 53:53/udp \
  -p 3000:3000/tcp \
  adguard/adguardhome
```

### Configure

1. Access setup wizard at `http://192.168.1.51:3000`
2. Set admin password
3. Add blocklists:
   - **OISD Big:** `https://big.oisd.nl`
   - **HaGeZi Multi Pro:** `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/pro.txt`
   - **Steven Black:** `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts`
4. **Router config:** Set DNS server to `192.168.1.51` — all devices now use AdGuard

---

## Pi 5: WireGuard

### Install

```bash
docker run -d --name wireguard \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  -e PUID=1000 -e PGID=1000 \
  -e TZ=America/Chicago \
  -e SERVERURL=auto \
  -e PEERS=phone,laptop \
  -e PEERDNS=192.168.1.51 \
  -p 51820:51820/udp \
  -v /opt/wireguard:/config \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --restart=always \
  linuxserver/wireguard
```

### Configure

1. Router: Forward UDP port `51820` to `192.168.1.51`
2. Find client configs:
   ```bash
   docker exec wireguard cat /config/peer_phone/peer_phone.conf
   ```
3. Scan QR code on phone or copy config to laptop WireGuard client
4. `PEERDNS=192.168.1.51` routes DNS through AdGuard even when remote

---

## Pi 5: Uptime Kuma

### Install

```bash
docker run -d --name uptime-kuma \
  --restart=always \
  -p 3001:3001 \
  -v /opt/uptime-kuma:/app/data \
  louislam/uptime-kuma
```

### Configure

1. Access at `http://192.168.1.51:3001`
2. Add monitors for each service:
   - Jellyfin: `http://192.168.1.50:8096`
   - Unraid: `http://192.168.1.50`
   - Sonarr: `http://192.168.1.50:8989`
   - Radarr: `http://192.168.1.50:7878`
   - AdGuard: `http://192.168.1.51:3000`
   - qBittorrent: `http://192.168.1.50:8080`
3. Optional: Set up notifications (Discord webhook, email, Telegram)

---

## Service Ports Quick Reference

| Service | Host | Port | URL |
|---|---|---|---|
| Unraid WebUI | M920q | 80 | `http://192.168.1.50` |
| Jellyfin | M920q | 8096 | `http://192.168.1.50:8096` |
| Sonarr | M920q | 8989 | `http://192.168.1.50:8989` |
| Radarr | M920q | 7878 | `http://192.168.1.50:7878` |
| Prowlarr | M920q | 9696 | `http://192.168.1.50:9696` |
| qBittorrent | M920q | 8080 | `http://192.168.1.50:8080` |
| Bazarr | M920q | 6767 | `http://192.168.1.50:6767` |
| Immich | M920q | 2283 | `http://192.168.1.50:2283` |
| AdGuard Home | Pi 5 | 3000 | `http://192.168.1.51:3000` |
| WireGuard | Pi 5 | 51820 | UDP only |
| Uptime Kuma | Pi 5 | 3001 | `http://192.168.1.51:3001` |
| Portainer | Pi 5 | 9443 | `https://192.168.1.51:9443` |
