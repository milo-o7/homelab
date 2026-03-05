# 10-Inch Mini Rack Homelab

> A budget-friendly, bedroom-quiet homelab built around a 10-inch GeeekPi rack, Lenovo M920q "ThinkNAS," and Raspberry Pi 5.

![Budget](https://img.shields.io/badge/budget-%24950--1%2C070-blue)
![Rack](https://img.shields.io/badge/rack-GeeekPi%2012U%20T2-orange)
![OS](https://img.shields.io/badge/OS-Unraid-green)
![Storage](https://img.shields.io/badge/storage-2x%204TB%20WD%20Red%20Plus-red)

```
┌──────────────────────────────────┐
│ U1-2   DeskPi 7.84" Screen (2U) │
├──────────────────────────────────┤
│ U3     Patch Panel + Brush Strip │
├──────────────────────────────────┤
│ U4     TP-Link Switch (1U)       │
├──────────────────────────────────┤
│ U5     Lenovo M920q (1U)         │
├──────────────────────────────────┤
│ U6     Raspberry Pi 5 (1U)       │
├──────────────────────────────────┤
│ U7-8   3D Printed HDD Cage (2U) │
├──────────────────────────────────┤
│ U9-10  Expansion (2U free)       │
├──────────────────────────────────┤
│ U11    Tupavco PDU (1U)          │
├──────────────────────────────────┤
│ U12    Cable Routing             │
├──────────────────────────────────┤
│ FLOOR  Tripp Lite UPS (0U)      │
└──────────────────────────────────┘
```

---

## Table of Contents

- [Overview](#overview)
- [Hardware](#hardware)
- [Rack Layout](#rack-layout)
- [Network Diagram](#network-diagram)
- [Software Stack](#software-stack)
- [3D Printed Parts](#3d-printed-parts)
- [Build Guide](#build-guide)
- [Shopping List](#shopping-list)
- [References](#references)

---

## Overview

This homelab is designed for a college apartment with three hard constraints:

| Constraint | Solution |
|---|---|
| **Small footprint** | 10-inch mini rack sits on a desk |
| **Near-silent** | Passive/quiet components, no server fans |
| **Low power draw** | ~65W total idle (M920q + Pi + switch) |

**What it does:**
- Media server (Jellyfin) with automated downloads (Sonarr/Radarr)
- Network-wide ad blocking (AdGuard Home)
- Photo backup (Immich — self-hosted Google Photos)
- VPN for remote access (WireGuard)
- Service monitoring (Uptime Kuma)

---

## Hardware

### Compute

| Component | Specs | Role | Price |
|---|---|---|---|
| **Lenovo M920q Tiny** | i5-8500T, 6C/6T, 16GB DDR4, 256GB NVMe | Main server (Unraid) | ~$130-190 |
| **Raspberry Pi 5 8GB** | Quad A76, 8GB LPDDR4X | Always-on services | $125 |

#### Why M920q over N100 mini PCs?

The PCIe riser slot is the killer feature. It accepts an ASM1166 M.2-to-SATA controller, adding 6 native SATA III ports — no case mods, no USB adapters.

<details>
<summary>Full comparison: M920q vs Beelink S12 Pro (N100)</summary>

| | M920q (i5-8500T) | Beelink S12 Pro (N100) |
|---|---|---|
| Idle power | ~7-10W | ~5-10W |
| Load power | ~45W | ~25W |
| Multi-thread perf | **85% faster** | Baseline |
| Perf/watt | ~226 pts/W | ~219 pts/W |
| HEVC 10-bit transcode | Yes | Yes |
| RAM | Up to 32GB dual-channel | 16GB single-channel |
| PCIe expansion | **Riser slot** | None |
| Build quality | Enterprise metal | Plastic |
| Price | ~$130-190 refurb | ~$159-169 new |

**Key insight:** The N100 efficiency advantage is a myth. Both idle at 7-10W. The 8th gen draws more under load but delivers 85% more multi-thread performance — actually *better* performance per watt.

</details>

### Storage

```
M920q                          3D Printed HDD Cage (U7-8)
┌─────────────┐                ┌─────────────────────┐
│  M.2 NVMe   │  cache/docker  │  WD Red Plus 4TB    │
│  256GB SSD   │                │  ┌─────────────────┐│
│              │                │  │ Drive 1 (array) ││
│  ASM1166     │ SATA cables   │  └─────────────────┘│
│  (PCIe riser)├───────────────►  ┌─────────────────┐│
│  6x SATA III │                │  │ Drive 2 (parity)││
└─────────────┘                │  └─────────────────┘│
                                └─────────────────────┘
                                Powered by 12V brick
                                + DC-ATX board
```

- **ASM1166** M.2-to-SATA controller (~$25) — sits in the M920q's PCIe riser slot
- **2x WD Red Plus 4TB** (WD40EFPX) — NAS-rated CMR drives, 5400 RPM
- **12V power brick** (10A+) + **RGEEK DC-ATX board** — provides SATA power connectors
- Drives mount in a **3D printed cage** bolted to the rack rails

### Networking

| Component | Specs | Price |
|---|---|---|
| **TP-Link TL-SG108** | 8-port unmanaged gigabit | $18 |
| **GeeekPi Patch Panel** | 12-port CAT6, 0.5U | $16 |

### Power

| Component | Specs | Price |
|---|---|---|
| **Tupavco TP1713 PDU** | 1U, 4-outlet, native 10" ears | $60 |
| **Tripp Lite BC600R UPS** | 600VA/300W, sits on rack floor | ~$97-130 |

### The Rack

**GeeekPi RackMate T2** — 12U, 10-inch, 260mm depth, $159

All GeeekPi/DeskPi accessories (shelves, patch panels, PDUs) are cross-compatible across the T-series lineup.

---

## Rack Layout

The layout follows community best practices from [geerlingguy/mini-rack](https://github.com/geerlingguy/mini-rack):

- Heavy items (UPS) at the bottom for stability
- Networking near the top to minimize patch cable runs
- Compute in the middle for accessibility
- Storage below compute (short SATA cable runs, weight adds stability)
- PDU near the bottom (short run to UPS)
- Screen at the top (visible at desk eye level)

```
GeeekPi 12U 10-Inch Rack (T2)
260mm depth | 210mm safe width

         FRONT VIEW                              REAR VIEW
┌────────────────────────────┐        ┌────────────────────────────┐
│  ┌──────────────────────┐  │        │                            │
│  │  7.84" Touch Screen  │  │  U1-2  │  HDMI + USB-C cables      │
│  │  1280 x 400 LCD      │  │        │  route down to M920q      │
│  └──────────────────────┘  │        │                            │
├────────────────────────────┤        ├────────────────────────────┤
│  [Patch Panel 12x CAT6]   │  U3    │  Wall ethernet entry       │
│  [////Brush Strip/////]    │        │  via brush strip           │
├────────────────────────────┤        ├────────────────────────────┤
│  ┌──────────────────────┐  │        │                            │
│  │ TP-Link TL-SG108     │  │  U4   │  Patch cables up to panel  │
│  └──────────────────────┘  │        │                            │
├────────────────────────────┤        ├────────────────────────────┤
│  ┌──────────────────────┐  │        │  ┌──SATA cables──┐        │
│  │  Lenovo M920q        │  │  U5   │  │  route down    │        │
│  │  i5-8500T | Unraid   │  │        │  └───────────────┘        │
│  └──────────────────────┘  │        │                            │
├────────────────────────────┤        ├────────────────────────────┤
│  ┌─────────┐               │        │                            │
│  │  Pi 5   │  SBC Shelf    │  U6   │  Ethernet + USB-C power   │
│  └─────────┘               │        │                            │
├────────────────────────────┤        ├────────────────────────────┤
│  ┌──────────────────────┐  │        │                            │
│  │  HDD Cage            │  │  U7-8 │  SATA data from M920q     │
│  │  2x WD Red Plus 4TB  │  │        │  12V power from brick     │
│  └──────────────────────┘  │        │                            │
├────────────────────────────┤        ├────────────────────────────┤
│        (expansion)         │  U9   │                            │
├────────────────────────────┤        ├────────────────────────────┤
│        (expansion)         │  U10  │                            │
├────────────────────────────┤        ├────────────────────────────┤
│  ┌──────────────────────┐  │        │                            │
│  │  Tupavco TP1713 PDU  │  │  U11  │  6ft cord exits to UPS    │
│  └──────────────────────┘  │        │                            │
├────────────────────────────┤        ├────────────────────────────┤
│    (cable routing)         │  U12  │  Power cables route up     │
├────────────────────────────┤        ├────────────────────────────┤
│  ╔══════════════════════╗  │        │                            │
│  ║ Tripp Lite BC600R    ║  │ FLOOR │  UPS sits below rails     │
│  ║ 600VA UPS            ║  │  (0U) │  262mm W fits outer shell │
│  ╚══════════════════════╝  │        │                            │
└────────────────────────────┘        └────────────────────────────┘

U Budget: 10U used + 2U free | UPS: 0U (floor)
```

### Cable Routing

| Cable | From | To | Length Needed |
|---|---|---|---|
| SATA data (x2) | M920q rear (U5) | HDD cage (U7-8) | 12-18" |
| Ethernet patches | Switch (U4) | Patch panel (U3) | 6-12" |
| HDMI | M920q (U5) | Screen (U1-2) | ~3 ft |
| USB-C | M920q (U5) | Screen (U1-2) | ~3 ft |
| 12V DC | Power brick (floor) | HDD cage (U7-8) | ~4 ft |
| Power cords | PDU (U11) | All devices | Routed up rear rail |

---

## Network Diagram

```
                    ┌──────────────┐
                    │   Internet   │
                    └──────┬───────┘
                           │
                    ┌──────┴───────┐
                    │  ISP Router  │  (elsewhere in apartment)
                    │  WiFi + DHCP │
                    └──────┬───────┘
                           │
              Wall ethernet port (bedroom)
                           │
         ┌─────────────────┴──────────────────┐
         │        GeeekPi Patch Panel          │
         │        12-port CAT6 (U3)            │
         └─────────────────┬──────────────────┘
                           │ short patch cables
         ┌─────────────────┴──────────────────┐
         │      TP-Link TL-SG108 Switch        │
         │      8-port gigabit (U4)            │
         └──┬──────────┬──────────┬────────────┘
            │          │          │
     ┌──────┴───┐ ┌────┴────┐    │
     │  M920q   │ │  Pi 5   │    │ (future devices)
     │ .1.50    │ │ .1.51   │    │
     │ Unraid   │ │ AdGuard │    │
     │ Jellyfin │ │ WireGrd │    │
     │ Sonarr   │ │ Uptime  │    │
     │ Radarr   │ │ Kuma    │    │
     └──────────┘ └─────────┘    │

WiFi Clients (via ISP Router):
  Phone ──► Jellyfin streaming (192.168.1.50:8096)
  Laptop ──► Unraid WebUI (192.168.1.50)
  All ──► DNS queries routed to Pi AdGuard (192.168.1.51)

Remote Access:
  Phone/Laptop ──► WireGuard VPN (UDP 51820) ──► Pi 5 ──► Full LAN access
```

### Key Router Settings

| Setting | Value | Purpose |
|---|---|---|
| Static DHCP lease | M920q = `192.168.1.50` | Server always reachable at same IP |
| Static DHCP lease | Pi 5 = `192.168.1.51` | DNS server always reachable |
| DNS server | `192.168.1.51` | Routes all DNS through AdGuard |
| Port forward | UDP `51820` -> `192.168.1.51` | WireGuard remote access |

---

## Software Stack

### M920q — Unraid OS

Unraid boots from a USB flash drive and manages the storage array with parity protection. All services run as Docker containers via the Community Applications plugin.

```
                    Unraid Web GUI (port 80/443)
                              │
              ┌───────────────┼───────────────┐
              │               │               │
        ┌─────┴─────┐  ┌─────┴─────┐  ┌──────┴──────┐
        │  Storage   │  │  Docker   │  │  Settings   │
        │  Manager   │  │  Manager  │  │  & Plugins  │
        └─────┬──────┘  └─────┬─────┘  └─────────────┘
              │               │
     ┌────────┴───────┐       │
     │ Parity Array   │       ├── Jellyfin     (media streaming)
     │ Drive 1: data  │       ├── Sonarr       (TV automation)
     │ Drive 2: parity│       ├── Radarr       (movie automation)
     │ Cache: NVMe    │       ├── Prowlarr     (indexer manager)
     └────────────────┘       ├── qBittorrent  (VPN-tunneled downloads)
                              ├── Bazarr       (subtitles)
                              └── Immich       (photo backup)
```

### Media Automation Pipeline

```
 1. Add show/movie          2. Search indexers       3. Download
    in Sonarr/Radarr  ────►    via Prowlarr    ────►    via qBittorrent
                                                        (through Mullvad VPN)
                                                              │
 6. Ready to stream    5. Subtitles added    4. File organized│
    on any device  ◄────  by Bazarr     ◄────  by Sonarr/  ◄─┘
    via Jellyfin                               Radarr

ISP sees: encrypted WireGuard traffic only
```

### Folder Structure (hardlinks, no duplicate files)

```
/mnt/user/data/
├── torrents/
│   ├── movies/
│   └── tv/
└── media/
    ├── movies/
    ├── tv/
    └── music/
```

> All apps must share the same `/data/` root. This enables hardlinks — Sonarr/Radarr "move" completed downloads instantly with zero disk usage overhead.

### Pi 5 — Always-On Services

| Service | Port | Purpose |
|---|---|---|
| **AdGuard Home** | 53 (DNS), 3000 (web) | Network-wide ad/tracker blocking |
| **WireGuard** | 51820 (UDP) | VPN tunnel for remote access |
| **Uptime Kuma** | 3001 | Monitors all services, sends alerts |

### Ad Blocking Notes

AdGuard blocks ads at the DNS level for every device on the network — phones, laptops, smart TVs, IoT, guests. No per-device setup needed.

**Recommended blocklists:** OISD (Big), HaGeZi Multi Pro, Steven Black

**YouTube ads** cannot be blocked via DNS (ads served from same domains as video). Use:
- Desktop: Firefox + uBlock Origin
- Android: ReVanced
- iPhone: Brave browser

---

## 3D Printed Parts

All parts printed in **PETG** (heat resistance required near electronics). Designed for 10-inch rack rail mounting.

### Required Prints

| Part | Model | Est. Weight | Est. Cost (Auburn Lab) |
|---|---|---|---|
| M920q 1U rack mount | [Printables #1040412](https://www.printables.com/model/1040412) | ~150-200g | ~$11-14 |
| HDD cage (2x 3.5") | [Printables #142325](https://www.printables.com/model/142325) | ~120-180g | ~$9-13 |
| **Total** | | **~300g** | **~$20-27** |

### Optional Prints

| Part | Model | Est. Cost |
|---|---|---|
| BC600R UPS 1U bracket | [Printables #1368936](https://www.printables.com/model/1368936) | ~$7 |
| Cable management plate | [Printables #1247474](https://www.printables.com/model/1247474) | ~$6 |
| 1U blank panels | Various | ~$4 each |

### Print Settings

- **Material:** PETG (1.24 g/cm3)
- **Infill:** 20-25% gyroid
- **Perimeters:** 3
- **Layer height:** 0.2mm
- **Hardware:** M3 screws (M3x6 or M3x10 countersunk), rubber anti-vibration grommets for HDD mount

### Alternative M920q Rack Mounts

| Source | Model |
|---|---|
| Printables | [#1190372](https://www.printables.com/model/1190372) — M920-specific |
| MakerWorld | [#1141511](https://makerworld.com/en/models/1141511) — ThinkCentre 10" rackmount |
| Blazin3D | Commercial 1U bracket (~$25-35) |
| Etsy | Various 10-inch ThinkCentre mounts ($25-45) |

---

## Build Guide

### Phase 1: Order Components

Buy in this order to start learning while waiting for parts:

1. **M920q + Pi 5** — start setting up Unraid and Pi OS
2. **Rack + accessories** — shelves, patch panel, brush strip
3. **ASM1166 + SATA cables + drives** — storage expansion
4. **PDU + UPS** — power management (can skip initially to save budget)

### Phase 2: M920q Prep

1. **BIOS setup** — Disable Secure Boot, set USB as first boot device
2. **ASM1166 firmware** — Flash ECS06 firmware from SilverStone's site (do on Windows before deploying)
3. **Install ASM1166** — Insert into PCIe riser slot, verify SATA ports are accessible from rear
4. **Flash Unraid** — Write to SanDisk 16GB USB drive using Unraid USB Creator
5. **Boot and license** — First boot, register Starter license ($49, up to 6 devices)

### Phase 3: Rack Assembly

1. Assemble T2 rack frame
2. Mount UPS on floor
3. Mount PDU at U11
4. Mount patch panel + brush strip at U3
5. Place switch on 1U shelf at U4
6. Mount M920q on 3D printed bracket at U5
7. Mount Pi 5 on SBC shelf at U6
8. Mount HDD cage at U7-8
9. Route all cables along rear rail, zip-tie

### Phase 4: Storage Setup

1. Connect SATA data cables from ASM1166 to drives in HDD cage
2. Connect 12V power brick to DC-ATX board, route SATA power to drives
3. In Unraid: assign Drive 1 as data, Drive 2 as parity
4. Start array, let parity build (~8-12 hours for 4TB)

### Phase 5: Network Configuration

1. Test wall ethernet port — plug laptop in, check for internet
2. Run cable from wall port to rack patch panel
3. Connect switch to patch panel, connect all devices to switch
4. Set static DHCP leases in router (M920q = .50, Pi = .51)
5. Set router DNS to Pi's IP for network-wide ad blocking

### Phase 6: Software Deployment

See [docs/software-setup.md](docs/software-setup.md) for detailed container configuration.

**Quick start order:**
1. Jellyfin (verify transcoding works)
2. AdGuard Home on Pi
3. Prowlarr + Sonarr + Radarr + qBittorrent (media pipeline)
4. WireGuard on Pi (remote access)
5. Uptime Kuma on Pi (monitoring)
6. Immich (photo backup)

---

## Shopping List

### Core Compute & Storage (~$535-595)

| Component | Price | Notes |
|---|---|---|
| Lenovo M920q i5-8500T 16GB 256GB | ~$130-250 | eBay refurb, confirm PCIe riser included |
| M920q PCIe riser (01AJ940) | ~$15 | Only if not included with M920q |
| ASM1166 M.2-to-SATA 6-port | ~$30 | RIITOP brand on Amazon/Newegg |
| 2x WD Red Plus 4TB (WD40EFPX) | ~$210-230 | NAS-rated CMR drives |
| SHNITPWR 12V 10A brick | ~$17 | Powers HDD cage |
| RGEEK DC-ATX board | ~$36 | Converts 12V to SATA power |
| SATA cables 18" (3-pack) | ~$8 | Cable Matters |
| SanDisk 16GB USB (Unraid boot) | ~$9 | |

### Raspberry Pi 5 (~$146-150)

| Component | Price |
|---|---|
| Raspberry Pi 5 8GB | $125 |
| Official 27W USB-C PSU | $13 |
| SanDisk 64GB microSD | ~$8-12 |

### Rack & Accessories (~$494-527)

| Component | Price |
|---|---|
| GeeekPi RackMate T2 12U | $159 |
| DeskPi 7.84" Touch Screen | ~$100 |
| Tripp Lite BC600R UPS | ~$97-130 |
| Tupavco TP1713 PDU | $60 |
| TP-Link TL-SG108 switch | $18 |
| GeeekPi 1U shelf (switch) | ~$17 |
| GeeekPi SBC shelf (Pi 5) | ~$16 |
| GeeekPi 12-port patch panel | ~$16 |
| GeeekPi brush strip | ~$11 |

### Cables & Small Parts (~$32)

| Component | Price |
|---|---|
| Cat6 patch cables 1ft (5-pack) | ~$9 |
| HDMI cable 1ft (2-pack) | ~$8 |
| USB-C cable 1ft (3-pack) | ~$8 |
| HDD screws + anti-vibe grommets | ~$7 |

### 3D Prints (~$24)

| Part | Cost |
|---|---|
| M920q 1U mount (PETG, ~175g) | ~$12.50 |
| HDD cage 2x 3.5" (PETG, ~150g) | ~$11.00 |

### Software

| Component | Price |
|---|---|
| Unraid Starter license | $49 (one-time) |
| Mullvad VPN | $5/month |

### Budget Summary

| Category | Subtotal |
|---|---|
| Core compute & storage | ~$535-595 |
| Raspberry Pi 5 | ~$146-150 |
| Rack & accessories | ~$494-527 |
| Cables & small parts | ~$32 |
| 3D prints | ~$24 |
| Software | $49 + $5/mo |
| **Grand total** | **~$1,230-1,380** |

<details>
<summary>Ways to hit ~$1,000</summary>

1. **Skip the screen** (-$100) — manage via SSH/web browser, add later
2. **Skip the PDU** (-$60) — use a $10 power strip outside the rack
3. **Find a cheaper M920q** — 8GB units run ~$130-150, add RAM later
4. **Skip the UPS initially** (-$100) — add after confirming stable apartment power
5. All four = **saves ~$360**, bringing total to **~$870-1,020**

</details>

---

## Component Dimensions & Fitment

All components verified against the T2's **260mm depth** and **210mm safe width**.

| Component | W x D x H (mm) | U Height | Fits? | Mount |
|---|---|---|---|---|
| Lenovo M920q | 179 x 183 x 34.5 | 1U | Yes | 3D printed bracket |
| DeskPi Screen | 254 x ~35 x 88.9 | 2U | Yes | Native rack ears |
| GeeekPi Patch Panel | 254 x ~50 x 23 | 0.5U | Yes | Native rack ears |
| TP-Link TL-SG108 | 158 x 101 x 25 | 1U (shelf) | Yes | GeeekPi 1U shelf |
| Tupavco TP1713 PDU | 254 x 44 x 44 | 1U | Yes (tight) | Native rack ears |
| Tripp Lite BC600R | 262 x 180 x 58 | 0U | Floor | Below rails |
| GeeekPi SBC Shelf | 254 x 200 x 44 | 1U | Yes | Native rack ears |
| 3D Printed HDD Mount | ~236 x ~200 x ~90 | ~2U | Yes | 3D printed ears |
| WD Red Plus 4TB | 101.6 x 147 x 26.1 | In cage | Yes | M3 screws + grommets |
| Raspberry Pi 5 | 56 x 85 x 17 | On shelf | Yes | SBC shelf |

---

## References

- **[geerlingguy/mini-rack](https://github.com/geerlingguy/mini-rack)** — Jeff Geerling's community guide for 10-inch mini racks
- **Cat Nest build (Issue #258)** — Same HBA-in-PCIe approach with 6x WD Red Plus
- **ThinkNAS community** — M920q + ASM1166 storage expansion pattern
- **[Unraid Documentation](https://docs.unraid.net/)** — Official Unraid setup guides

---

## License

This project documentation is released under [MIT](LICENSE). Hardware choices and 3D print models are from their respective creators — see links for their licenses.
