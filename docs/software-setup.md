# Software Setup Guide

Step-by-step configuration for all Docker containers on the M920q (Unraid).

---

## Table of Contents

- [Unraid Initial Setup](#unraid-initial-setup)
- [Storage Configuration](#storage-configuration)
- [Tailscale](#tailscale)
- [Jellyfin](#jellyfin)
- [Arr Stack (Sonarr, Radarr, Prowlarr)](#arr-stack)
- [Flaresolverr](#flaresolverr)
- [qBittorrent + VPN](#qbittorrent--vpn)
- [Bazarr](#bazarr)
- [Immich](#immich)
- [Nginx Proxy Manager](#nginx-proxy-manager)
- [Homepage](#homepage)
- [Scrutiny](#scrutiny)
- [Watchtower](#watchtower)
- [AdGuard Home](#adguard-home)
- [WireGuard](#wireguard)
- [Uptime Kuma](#uptime-kuma)

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

## Tailscale

**Purpose:** Mesh VPN that works behind CGNAT — remote access without port forwarding

Apartment internet often uses carrier-grade NAT, making traditional WireGuard port forwarding impossible. Tailscale creates an encrypted mesh network between all your devices with zero router configuration.

### Install on Unraid (M920q)

1. **Apps** > search "Tailscale" > install the official `tailscale/tailscale` container
2. Or install via **Plugins** > search "Tailscale" (Unraid has a native plugin)
3. Authenticate at the URL shown in the container logs
4. In Tailscale admin console: enable **Subnet Routes** for `192.168.1.0/24` to access your whole LAN remotely

### Install on Phone/Laptop

1. Download Tailscale app (iOS, Android, macOS, Windows, Linux)
2. Sign in with same account
3. All devices can now reach each other by Tailscale IP or hostname

### Key Settings

- **MagicDNS:** Enabled — access M920q as `m920q` instead of an IP
- **Exit node:** Enable on M920q to route all traffic through your homelab when remote (like a traditional VPN)
- **ACLs:** Default allows all devices to see each other — fine for personal use

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

## Flaresolverr

**Purpose:** Solves Cloudflare CAPTCHAs so Prowlarr can access protected indexer sites

Many torrent indexers use Cloudflare anti-bot challenges. Prowlarr can't solve CAPTCHAs on its own — without Flaresolverr, half your indexers will randomly stop working.

### Install

1. **Apps** > search "Flaresolverr" > install `ghcr.io/flaresolverr/flaresolverr`
2. Port: `8191`
3. No path mappings needed — it runs a headless browser internally

### Connect to Prowlarr

1. In Prowlarr: **Settings > Indexers > Add** (under Indexer Proxies)
2. Select **FlareSolverr**
3. Tag: `flaresolverr`
4. Host: `http://flaresolverr:8191`
5. Apply the `flaresolverr` tag to any indexer that uses Cloudflare

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

## Nginx Proxy Manager

**Purpose:** Reverse proxy — access services by name (e.g., `jellyfin.home`) instead of `192.168.1.50:8096`

### Install

1. **Apps** > search "Nginx Proxy Manager" > install `jc21/nginx-proxy-manager`
2. Ports: `80` (HTTP), `443` (HTTPS), `81` (admin panel)
3. Config path: `/mnt/user/appdata/nginx-proxy-manager`
4. Default login: `admin@example.com` / `changeme` (change immediately)

### Configure

1. Access admin at `http://192.168.1.50:81`
2. For each service, add a **Proxy Host:**
   - Domain: `jellyfin.home` (or `jellyfin.yourdomain.com` if you own a domain)
   - Scheme: `http`
   - Forward hostname: `192.168.1.50`
   - Forward port: `8096`
   - Enable "Block Common Exploits"
3. For SSL with a real domain: use the built-in Let's Encrypt integration (free certs, auto-renewal)

### Local DNS Integration

For `.home` domains to work locally, add DNS rewrites in AdGuard Home:
- `*.home` -> `192.168.1.50`

Now `jellyfin.home`, `sonarr.home`, etc. all resolve to the M920q, and Nginx Proxy Manager routes them to the correct port.

---

## Homepage

**Purpose:** Clean dashboard showing all services with live status, system stats, and widgets

### Install

1. **Apps** > search "Homepage" > install `ghcr.io/gethomepage/homepage`
2. Port: `3000` (change to `3100` if conflicting with AdGuard)
3. Config path: `/mnt/user/appdata/homepage`

### Configure

Edit `/mnt/user/appdata/homepage/services.yaml`:

```yaml
- Media:
    - Jellyfin:
        href: http://192.168.1.50:8096
        icon: jellyfin.png
        widget:
          type: jellyfin
          url: http://192.168.1.50:8096
          key: YOUR_API_KEY

    - Sonarr:
        href: http://192.168.1.50:8989
        icon: sonarr.png
        widget:
          type: sonarr
          url: http://192.168.1.50:8989
          key: YOUR_API_KEY

    - Radarr:
        href: http://192.168.1.50:7878
        icon: radarr.png
        widget:
          type: radarr
          url: http://192.168.1.50:7878
          key: YOUR_API_KEY

- Infrastructure:
    - AdGuard:
        href: http://192.168.1.50:3000
        icon: adguard-home.png
        widget:
          type: adguard
          url: http://192.168.1.50:3000
          username: YOUR_USER
          password: YOUR_PASS

    - Scrutiny:
        href: http://192.168.1.50:8082
        icon: scrutiny.png
        widget:
          type: scrutiny
          url: http://192.168.1.50:8082
```

Homepage supports widgets for nearly every homelab service — it pulls live data (active streams, queue counts, disk health, DNS queries blocked) directly into the dashboard.

---

## Scrutiny

**Purpose:** Monitors SMART data on your WD Red Plus drives — alerts before drive failure

With only 2 drives (data + parity), a silent drive failure can be catastrophic. Scrutiny checks temperatures, reallocated sectors, power-on hours, and gives each drive a health grade.

### Install

1. **Apps** > search "Scrutiny" > install `ghcr.io/analogj/scrutiny`
2. Port: `8082`
3. Config path: `/mnt/user/appdata/scrutiny`
4. **Critical:** Pass through `/dev/disk` so Scrutiny can read SMART data:
   - Device: `/dev/disk` -> `/dev/disk`
   - Also pass: `/run/udev` -> `/run/udev:ro`
5. The container needs `--cap-add=SYS_RAWIO` for direct disk access

### Configure

1. Access at `http://192.168.1.50:8082`
2. Drives should auto-detect — verify both WD Red Plus drives appear
3. SMART scan runs automatically on a schedule (default: every 24 hours)
4. Set up notifications: **Settings > Notifications** — supports Discord, email, Pushover, etc.

### What to Watch

| Metric | Warning Sign |
|---|---|
| Reallocated Sector Count | Any non-zero value — drive is remapping bad sectors |
| Current Pending Sector | Sectors waiting to be remapped — imminent failure indicator |
| Temperature | Sustained >45C for WD Red Plus — check Noctua fan |
| Power-On Hours | >30,000 hours — consider proactive replacement |

---

## Watchtower

**Purpose:** Automatically updates Docker containers when new images are released

### Install

1. **Apps** > search "Watchtower" > install `containrrr/watchtower`
2. No port needed — Watchtower has no web UI
3. Pass Docker socket: `/var/run/docker.sock` -> `/var/run/docker.sock`

### Configure

Key environment variables:

| Variable | Value | Purpose |
|---|---|---|
| `WATCHTOWER_SCHEDULE` | `0 0 4 * * *` | Check for updates at 4 AM daily |
| `WATCHTOWER_CLEANUP` | `true` | Remove old images after updating |
| `WATCHTOWER_NOTIFICATIONS` | `shoutrrr` | Enable notifications |
| `WATCHTOWER_NOTIFICATION_URL` | `discord://webhook_id/token` | Discord webhook for update alerts |

> Watchtower will update ALL containers by default. To exclude a container from auto-updates (e.g., Unraid itself), add the label `com.centurylinklabs.watchtower.enable=false` to that container.

---

## AdGuard Home

**Purpose:** Network-wide DNS ad blocker — blocks ads and trackers for every device on the network

### Install

1. Go to **Apps** tab, search "AdGuard Home"
2. Install the `adguard/adguardhome` container
3. Key settings:
   - **DNS port:** `53` (TCP and UDP)
   - **Web UI port:** `3000`
   - **Config path:** `/mnt/user/appdata/adguardhome/conf`
   - **Work path:** `/mnt/user/appdata/adguardhome/work`

### Configure

1. Access setup wizard at `http://192.168.1.50:3000`
2. Set admin password
3. Add blocklists:
   - **OISD Big:** `https://big.oisd.nl`
   - **HaGeZi Multi Pro:** `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/pro.txt`
   - **Steven Black:** `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts`
4. **Router config:** Set DNS server to `192.168.1.50` — all devices now use AdGuard

---

## WireGuard

**Purpose:** VPN server for remote access to the local network

### Install

1. Go to **Apps** tab, search "WireGuard" > install the `linuxserver/wireguard` container
2. Key template settings:
   - **Port:** `51820` (UDP)
   - **Config path:** `/mnt/user/appdata/wireguard`
   - **Key environment variables:**

| Variable | Value | Purpose |
|---|---|---|
| `SERVERURL` | `auto` | Auto-detects your public IP |
| `PEERS` | `phone,laptop` | Creates a config per client |
| `PEERDNS` | `192.168.1.50` | Routes DNS through AdGuard on the same host |
| `TZ` | `America/Chicago` | Timezone |

3. Under **Extra Parameters**, add: `--cap-add=NET_ADMIN --cap-add=SYS_MODULE --sysctl="net.ipv4.conf.all.src_valid_mark=1"`

### Configure

1. Router: Forward UDP port `51820` to `192.168.1.50`
2. Find client configs via Unraid terminal:
   ```bash
   docker exec wireguard cat /config/peer_phone/peer_phone.conf
   ```
3. Scan QR code on phone or copy config to laptop WireGuard client
4. `PEERDNS=192.168.1.50` routes DNS through AdGuard even when remote

---

## Uptime Kuma

**Purpose:** Self-hosted uptime monitoring for all services

### Install

1. Go to **Apps** tab, search "Uptime Kuma" > install the `louislam/uptime-kuma` container
2. Key settings:
   - **Port:** `3001`
   - **Config path:** `/mnt/user/appdata/uptime-kuma`

### Configure

1. Access at `http://192.168.1.50:3001`
2. Add monitors for each service:
   - Jellyfin: `http://192.168.1.50:8096`
   - Unraid: `http://192.168.1.50`
   - Sonarr: `http://192.168.1.50:8989`
   - Radarr: `http://192.168.1.50:7878`
   - AdGuard: `http://192.168.1.50:3000`
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
| Flaresolverr | M920q | 8191 | `http://192.168.1.50:8191` |
| qBittorrent | M920q | 8080 | `http://192.168.1.50:8080` |
| Bazarr | M920q | 6767 | `http://192.168.1.50:6767` |
| Immich | M920q | 2283 | `http://192.168.1.50:2283` |
| Nginx Proxy Manager | M920q | 81 | `http://192.168.1.50:81` |
| Homepage | M920q | 3100 | `http://192.168.1.50:3100` |
| Scrutiny | M920q | 8082 | `http://192.168.1.50:8082` |
| Watchtower | M920q | — | No web UI (runs in background) |
| Tailscale | M920q | — | Managed via `tailscale.com/admin` |
| AdGuard Home | M920q | 3000 | `http://192.168.1.50:3000` |
| WireGuard | M920q | 51820 | UDP only (backup VPN) |
| Uptime Kuma | M920q | 3001 | `http://192.168.1.50:3001` |
