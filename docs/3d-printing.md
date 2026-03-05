# 3D Printing Guide

All custom rack-mountable parts for the 10-inch mini rack, printed in PETG at the Auburn University Print Lab.

---

## Print Lab Details

- **Location:** Auburn University campus print lab
- **Service fee:** $2.00 flat per job
- **PETG filament:** $0.06/gram
- **Turnaround:** Varies by lab load — submit early

---

## Required Parts

### 1. M920q 1U Rack Mount

Mounts the Lenovo ThinkCentre M920q Tiny directly to 10-inch rack rails at U5.

| Detail | Value |
|---|---|
| **Primary STL** | [Printables #1040412](https://www.printables.com/model/1040412) — "Lenovo ThinkCentre Tiny 10'' Rack Mount" by Tim |
| **Compatibility** | M720q / M920q / P330 Tiny |
| **Features** | Includes optional keystone port cutouts |
| **Est. weight** | ~150-200g |
| **Est. cost** | ~$11-14 ($2 fee + $9-12 filament) |

**Alternative options:**
| Source | Model | Notes |
|---|---|---|
| Printables | [#1190372](https://www.printables.com/model/1190372) | M920-specific design |
| MakerWorld | [#1141511](https://makerworld.com/en/models/1141511) | Generic ThinkCentre mount |
| MakerWorld | [#1188439](https://www.printables.com/model/1188439) | Parametric design (adjustable) |
| DeskPi | [GitHub](https://github.com/DeskPi-Team/3DPrint-Models) | Official DeskPi models |

**Commercial alternatives (no printing needed):**
- Blazin3D: 10" x 6.8" x 1.75" (exactly 1U) — ~$25-35
- Etsy: Various 10-inch ThinkCentre Tiny mounts — $25-45

### 2. HDD Cage (2x 3.5")

Mounts two WD Red Plus 4TB drives directly to 10-inch rack rails at U7-8.

| Detail | Value |
|---|---|
| **Primary STL** | [Printables #142325](https://www.printables.com/model/142325) — "10'' rack harddrive mount" by Karel |
| **Capacity** | 2x 3.5" drives |
| **Occupies** | ~1-2U |
| **Est. weight** | ~120-180g |
| **Est. cost** | ~$9-13 ($2 fee + $7-11 filament) |

**Alternative HDD cage options:**
| Source | Model | Notes |
|---|---|---|
| Printables | [#1320021](https://www.printables.com/model/1320021) | Caddy-less, permanent install |
| Printables | [#1152538](https://www.printables.com/model/1152538) | 2x 3.5" by Mr. Hrundeel |
| MakerWorld | ThinkNAS 6x HDD Enclosure | Purpose-built for M920q |
| MakerWorld | 10" Rack 1U 2x 3.5" HDD HOT SWAP | Hot-swap capable |
| MakerWorld | Stackable 10" 1U Rack 3.5" HDD Cage | Expandable |
| Etsy | DeskPi RackMate dual 3.5" HDD shelves | ~$25-45 each |

---

## Optional Parts

### UPS Rail Bracket

Mount the Tripp Lite BC600R on rack rails instead of the floor.

| Detail | Value |
|---|---|
| **STL** | [Printables #1368936](https://www.printables.com/model/1368936) |
| **Est. weight** | ~80g |
| **Est. cost** | ~$7 |

### Cable Management Plate

| Detail | Value |
|---|---|
| **STL** | [Printables #1247474](https://www.printables.com/model/1247474) |
| **Est. weight** | ~60g |
| **Est. cost** | ~$6 |

### 1U Blank Panels

Fill empty rack units for aesthetics and airflow management.

| Detail | Value |
|---|---|
| **STL** | Various on Printables/MakerWorld |
| **Est. weight** | ~30g each |
| **Est. cost** | ~$4 each |

### ASM1166 Mounting Bracket

Bracket for securing the ASM1166 card inside the M920q.

| Detail | Value |
|---|---|
| **STL** | [MakerWorld #1246251](https://makerworld.com/en/models/1246251) |
| **Est. weight** | ~10g |
| **Est. cost** | ~$3 |

---

## Print Settings

All parts should be printed with these settings for structural integrity near electronics:

| Setting | Value | Why |
|---|---|---|
| **Material** | PETG | Heat resistance (drives and PSU generate warmth) |
| **Infill** | 20-25% gyroid | Good strength-to-weight ratio |
| **Perimeters** | 3 | Structural shell strength |
| **Layer height** | 0.2mm | Standard, good surface finish |
| **Density** | 1.24 g/cm3 (PETG) | For weight/cost estimation |

---

## Hardware Needed

| Hardware | Quantity | Purpose |
|---|---|---|
| M3x6 or M3x10 countersunk screws | 8 | HDD cage drive mounting |
| M3 screws | 4 | M920q bracket to rails |
| Rubber anti-vibration grommets | 8 | HDD mount screws (primary vibration mitigation) |
| 10-32 rack screws | 8-12 | Bracket to rack rail attachment |

---

## Before Printing Checklist

1. Download all STL files from Printables/MakerWorld
2. Open in **PrusaSlicer** or **Bambu Studio**
3. Set material to PETG, infill to 20% gyroid, 0.2mm layer height
4. Verify dimensions:
   - M920q bracket: clearance for 179mm x 183mm chassis
   - HDD cage: fits 2x WD Red Plus (101.6mm x 147mm x 26.1mm each)
   - Both: rail hole spacing matches 10-inch standard (236.525mm between screw holes)
5. Note exact gram count from slicer — multiply by $0.06 + $2.00 for Auburn pricing
6. Submit to Auburn print lab (they may run the estimate for free before committing)
