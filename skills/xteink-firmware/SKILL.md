---
name: xteink Firmware
description: 'Use when the user needs: Build, flash, optimize, and manage the xteink_x4 ESP32-C3 e-paper firmware. Covers PlatformIO builds, OTA bank management, flash size analysis, release optimization, and device metadata. Trigger on xteink, trmnl, e-paper firmware, ESP32-C3 build, PlatformIO flash, OTA bank, firmware optimize.'
icon: icon.svg
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: xteink-firmware
  icon_style: craft-category-glyph-v1
---

# xteink_x4 Firmware Skill

## Device Quick Reference

- **MCU:** ESP32-C3 RISC-V, 80 MHz, 4 MB flash, 327 KB RAM
- **Display:** 4.26" e-paper 800x480 (bb_epaper, EP426_800x480)
- **Project:** `/Volumes/SHADOW/00-spaces/Personal/dev/clawd/workspaces/trmnl-x4/`
- **Also at:** `/Volumes/SHADOW/shadow-lab/apps/shadow-eye/`
- **Firmware:** v1.7.4, dual Arduino + ESP-IDF framework
- **Platform:** espressif32@6.12.0

## Build Commands

```bash
# Debug build (default, includes debug symbols)
pio run -e xteink_x4

# Release build (optimized for size, -Os)
pio run -e xteink_x4_release

# Flash via USB
pio run -e xteink_x4 -t upload

# Monitor serial output
pio device monitor -e xteink_x4

# Build + flash + monitor
pio run -e xteink_x4 -t upload && pio device monitor -e xteink_x4

# Clean build
pio run -e xteink_x4 -t clean

# Size analysis
pio run -e xteink_x4 -t size
```

## Partition Layout (4 MB flash)

```
NVS:      0x09000 (20 KB)  — preferences
OTA Data: 0x0E000 (8 KB)   — bank selector
Bank A:   0x10000 (1,875 KB) ota_0
Bank B:   0x1E0000 (1,875 KB) ota_1
SPIFFS:   0x3B0000 (256 KB)
Coredump: 0x3F0000 (64 KB)
```

## OTA Bank Management

- A/B dual-bank via Arduino `Update` class
- Triggered by API: `update_firmware=true` + `firmware_url`
- **Rollback is DISABLED** in debug builds
- Release build (`xteink_x4_release`) enables rollback flags
- OTA code in `src/bl.cpp` function `checkAndPerformFirmwareUpdate()`

## Pin Map

| Function | GPIO |
|----------|------|
| EPD SCK | 8 |
| EPD MOSI | 10 |
| EPD CS | 21 |
| EPD RST | 5 |
| EPD DC | 4 |
| EPD BUSY | 6 |
| Battery ADC | 0 |
| Button/Wake | 3 |
| MOSFET Power | 13 |

## Key Source Files

| File | Purpose |
|------|---------|
| `platformio.ini` | Build config, all board environments |
| `src/bl.cpp` | Main business logic, OTA, sleep, API |
| `src/display.cpp` | E-paper driver, rendering |
| `src/DEV_Config.h` | Pin definitions per board |
| `include/config.h` | FW version, preferences keys, constants |
| `include/pins.h` | Pin init interface |
| `min_spiffs.csv` | 4 MB partition table |
| `sdkconfig.xteink_x4` | ESP-IDF config |
| `scripts/git_version.py` | Git hash injection |

## xteink-Specific Code Paths

The `BOARD_XTEINK_X4` define gates these differences from TRMNL:
- **Pins:** Different EPD CS (21), BUSY (6), Battery (0), Interrupt (3)
- **Display:** Uses `PLANE_FALSE_DIFF` write mode (ghosting compensation)
- **Sleep:** Holds GPIO 13 MOSFET during deep sleep for battery power
- **Display profiles:** 3 profiles all using EP426_800x480 + 4GRAY

## Optimization Checklist

When optimizing flash usage:
1. Switch to `xteink_x4_release` (saves ~80 KB, 5.8%)
2. Check `CORE_DEBUG_LEVEL` is 0 (currently set)
3. Consider if `libsodium` can be excluded (5 MB archive, may be unused)
4. ESPAsyncWebServer is the largest app-level lib (13.5 MB archive)
5. CoAP, nghttp, asio, freemodbus are IDF components that may link unused
6. Enable OTA rollback in sdkconfig for production safety

## Flash Budget

| Build | Size | Usage | Headroom |
|-------|------|-------|----------|
| Debug | 1,384,106 B | 72.8% | 504 KB |
| Release | 1,304,200 B | 68.6% | 584 KB |
| Max per bank | 1,900,544 B | — | — |

**Warning threshold:** >85% (>1,615,000 B) — approaching danger zone for OTA


## Pipeline

```
Intent → Resolve target → Execute → Validate → Report
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Standard output | Normal use |
| `verbose` | Detailed diagnostics | Debugging |

## Artifact Routing

- Reports: `ShadowArchive/80-reports/`

## Contract

- Destructive operations require explicit operator confirmation
- Always dry-run before applying changes

