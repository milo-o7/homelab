# Hardware Research & Decisions

Detailed rationale behind every hardware decision, including alternatives researched and rejected.

---

## Mini PC Selection

### Winner: Lenovo ThinkCentre M920q Tiny

The M920q was chosen for one decisive reason: its **PCIe riser slot** cleanly solves the storage expansion problem that no other mini PC in this price range can.

The riser accepts an ASM1166 M.2-to-SATA controller card, adding 6 native SATA III ports. SATA cables route out the rear of the chassis to an external HDD cage — no case modifications required. This is a well-documented pattern in the homelab community, known as the "ThinkNAS" build.

### Alternatives Researched and Rejected

| Machine | Why Rejected |
|---|---|
| **Beelink S12 Pro (N100)** | No clean SATA expansion. Case is only 38mm tall — ASM1166 card physically won't fit with the lid closed. Would need permanent case modification or external USB adapters. |
| **Lenovo M700 Tiny (6th gen)** | M.2 slot is SATA-only (no NVMe). Intel QSV drivers deprecated on Linux for 6th gen Skylake. No HDMI output. |
| **Dell OptiPlex 3070 Micro (9th gen)** | $200 on FB Marketplace was overpriced vs $110-160 refurb on eBay. No PCIe riser slot like the M920q. |
| **Lenovo M910q (i5-6500T)** | Same 6th gen Skylake problems as M700. 8GB RAM, 250GB SSD at $157 renewed — worse value than M920q in every dimension. |
| **CWWK/Topton N100 NAS board** | Purpose-built with onboard SATA, but requires a full custom build (case, PSU, RAM, assembly). Board alone is $140-200. Blows the budget. |

### The N100 Efficiency Myth

Extensive research (including a 40+ hour community comparison guide) found that the efficiency gap between N100 and 8th gen Intel is negligible:

- Both idle at **~7-10W**
- Under load, the 8th gen draws more wattage but delivers **85% more multi-thread performance**
- Performance per watt: **~226 pts/W** (8th gen) vs **~219 pts/W** (N100)
- The N100's only real advantages: AV1 decode (most content is HEVC, not AV1) and newer Intel driver stack

---

## Storage Approach

### Winner: ASM1166 via PCIe Riser + 3D Printed HDD Cage

This is the "ThinkNAS" approach validated by the homelab community. The Cat Nest build (geerlingguy/mini-rack Issue #258) uses the same pattern with 6x WD Red Plus drives.

### How It Works

1. M920q boots Unraid from USB flash drive
2. Internal M.2 NVMe SSD serves as cache/Docker drive
3. ASM1166 controller card in PCIe riser adds 6x SATA III ports
4. SATA data cables route from riser out to 3D printed HDD cage below
5. Drives powered by external 12V brick + DC-ATX board

### ASM1166 Technical Details

- **Form factor:** M.2 2280 (80mm x 22mm x ~4mm)
- **Interface:** Designed for PCIe x2, works at x1 with no issues for HDDs
- **Bandwidth:** PCIe x1 = ~1 GB/s, plenty for 4+ spinning drives (each maxes ~180-260 MB/s)
- **Linux support:** Native `ahci` driver, no proprietary drivers needed
- **Firmware:** Some cards need one-time ECS06 flash from SilverStone's site — do on Windows first
- **Brands:** RIITOP, Sintech, 10Gtek on Amazon ($20-35)

### Storage Approaches Rejected

| Approach | Why Rejected |
|---|---|
| **USB multi-bay enclosure** (Mediasonic, ICY DOCK) | USB works but is less clean for Unraid array management. User specifically wants native SATA inside the rack. |
| **Synology NAS on shelf** | ~$300 diskless + separate OS. Redundant with Unraid. Adds complexity and power draw. |
| **5.25" bay hot swap cages** | Designed for desktop towers. Need 5.25" bay, internal SATA, internal PSU — mini PCs have none of these. |
| **19-inch rack** | Opens commercial DAS/NAS ecosystem but defeats the 10-inch desk-friendly goal. |

---

## Rack Selection

### Winner: GeeekPi RackMate T2 (12U)

| Model | Size | Depth | Price | Verdict |
|---|---|---|---|---|
| T0 | 4U | Open | $110 | Too small — screen alone is 2U |
| T1 | 8U | 200mm | $120 | Only 1U free after all components |
| T1 Plus | 8U | 260mm | $129 | Same U problem as T1 |
| TL1 | 10U | Open | ~$140 | No side panels |
| **T2** | **12U** | **260mm** | **$159** | **2U free for expansion, $39 premium worth it** |

---

## UPS Selection

### Winner: Tripp Lite BC600R

The originally planned **APC BE600M1** is **274mm wide** — it physically does not fit inside the T2 rack (236.5mm rail span, ~260mm outer shell). This was caught during dimension verification.

The BC600R at 262mm wide exceeds the rail span but fits within the outer shell, sitting on the rack floor below the lowest U position. Documented as a known-good fit in the Geerling community (Issue #1).

---

## Switch Upgrade Path

Starting with the TP-Link TL-SG108 ($18, unmanaged gigabit) and upgrading when needed:

| Switch | Ports | Features | Price | 10" Rack Mount |
|---|---|---|---|---|
| TP-Link TL-SG108E | 8x 1G | Managed, VLANs | ~$30 | Shelf |
| MikroTik CSS610-8G-2S+ | 8x 1G + 2x 10G SFP+ | Managed | ~$110 | RMK-2/10 kit |
| GiGaPlus GP-S25-0802P | 8x 2.5G PoE+ + 2x 10G SFP+ | PoE | $99 | 3D printed ears |

---

## 10-Inch Rack Standard

From the Geerling mini-rack project — the informal industry dimensions:

- **1U height:** 44.45mm (1.75")
- **Width between screw holes:** 236.525mm (9.312")
- **Max usable horizontal clearance:** 220mm (8.75")
- **Recommended safe working width:** 210mm (8.45")
- **Screw type:** 10-32 (GeeekPi standard)
- **Vertical hole spacing:** Same as standard 19-inch racks

All GeeekPi/DeskPi accessories are cross-compatible across T0/T1/T1 Plus/T2.
