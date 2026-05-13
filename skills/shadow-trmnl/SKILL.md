---
name: SHADOW TRMNL
description: 'Use when the user needs: ECHO — the Shadow Lab e-paper display surface. Full device control: push images via TurboQuant pipeline, manage playlists via Account API, check status, generate 800x480 renders, configure firmware. Embodies SP-015 absolute addressing mode. Trigger on: TRMNL, e-paper, ECHO, e-ink, trmnl display, kernel display, absolute display, push to display, update display, xteink, what''s on the display, check ECHO, shadow-light, display status.'
icon: icon.svg
metadata:
  depends_on: [xteink-firmware]
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-trmnl
  icon_style: craft-category-glyph-v1
---

# ECHO — Shadow Lab E-Paper Display

**Codename:** ECHO | **Product:** shadow-light | **Role:** Absolute surface (SP-015)

The physical embodiment of the Shadow Lab kernel state. E-ink medium — persistent, immutable between updates, no power needed to maintain image. The display that makes the shadow visible.

## Device Identity

| Field | Value |
|-------|-------|
| Codename | **ECHO** |
| Hardware | xteink_x4 ESP32-C3 RISC-V, 80-160 MHz, 4MB flash, 327KB RAM |
| Display | 4.26" e-paper, 800x480, 1-bit B/W + 2-bit (4-gray) |
| Controller | SSD1677 (Solomon Systech) |
| Driver | bb_epaper (EP426_800x480), PLANE_FALSE_DIFF write mode |
| Device ID | 41370 |
| Friendly ID | U1Y2AN |
| MAC | 88:56:A6:F0:50:94 |
| Firmware | 1.7.4 (1.7.8 available, ECHO mod built) |
| Account | clawd.ctrl@gmail.com (Agent Mac) |
| Refresh | 14 min (configurable 1-1440 min) |

## Credentials (Keychain-stored)

| Purpose | Keychain | Value |
|---------|----------|-------|
| Device Access Token | — | `1qsLJdoKJPtcyUTdTeqayg` |
| Account API Key | trmnl / trmnl-account-api | `user_crfhaacrc798tqf80km1n9du` |
| Webhook Image UUID | trmnl-shadow-lab / trmnl-webhook-image | `f5bc3d4e-ab85-495a-9f02-7d7c126b24ca` |
| Login Password | trmnl / clawd.ctrl@gmail.com | In keychain |

## Quick Commands

### Push image to ECHO (fastest path)
```bash
# TurboQuant pipeline — generate from live mesh data + push
python3 /Volumes/SHADOW/shadow-lab/scripts/echo-turboquant.py

# 4-gray dithered mode
python3 /Volumes/SHADOW/shadow-lab/scripts/echo-turboquant.py --4gray

# Delta mode (skip if unchanged)
python3 /Volumes/SHADOW/shadow-lab/scripts/echo-turboquant.py --delta

# Raw curl push (any 800x480 PNG)
curl -X POST "https://trmnl.com/api/plugin_settings/f5bc3d4e-ab85-495a-9f02-7d7c126b24ca/image" \
  -H "Content-Type: image/png" --data-binary @/path/to/image.png
```

### Check current display
```bash
# Peek without advancing playlist
curl -s "https://trmnl.app/api/display/current" \
  -H "Access-Token: 1qsLJdoKJPtcyUTdTeqayg" \
  -H "ID: 937793dbb2e716a7f8f273cde78ceb28" | python3 -m json.tool

# Download current image
IMG=$(curl -s "https://trmnl.app/api/display/current" \
  -H "Access-Token: 1qsLJdoKJPtcyUTdTeqayg" \
  -H "ID: 937793dbb2e716a7f8f273cde78ceb28" | python3 -c "import sys,json; print(json.load(sys.stdin)['image_url'])")
curl -sL "$IMG" -o /tmp/echo-current.png
```

### Manage playlist (Account API)
```bash
BEARER="Authorization: Bearer user_crfhaacrc798tqf80km1n9du"

# List playlist items
curl -s "https://trmnl.com/api/playlists/items" -H "$BEARER" | python3 -m json.tool

# Hide/show a plugin
curl -X PATCH "https://trmnl.com/api/playlists/items/{ID}" \
  -H "$BEARER" -H "Content-Type: application/json" -d '{"visible": false}'

# List all plugin settings
curl -s "https://trmnl.com/api/plugin_settings" -H "$BEARER" | python3 -m json.tool

# Get user profile
curl -s "https://trmnl.com/api/me" -H "$BEARER" | python3 -m json.tool
```

### Test Liquid templates (free, no auth)
```bash
curl -X POST "https://trmnl.com/api/markup" \
  -H "Content-Type: application/json" \
  -d '{"markup": "{{ title }} — {{ count }} nodes", "variables": {"title": "ECHO", "count": 7}}'
```

### Flash ECHO firmware (needs USB-C)
```bash
cd /Volumes/SHADOW/00-spaces/Personal/dev/clawd/workspaces/trmnl-x4
pio run -e xteink_x4 -t upload     # flash
pio device monitor -e xteink_x4     # serial monitor (115200 baud)
```

## Display Architecture

### Controller: SSD1677
- **SPI:** 20 MHz (GPIO 8=SCK, 10=MOSI, 21=CS, 5=RST, 4=DC, 6=BUSY)
- **Power:** GPIO 13 MOSFET hold during deep sleep
- **Button:** GPIO 3 (active LOW, wakes from deep sleep)
- **Battery ADC:** GPIO 0 (÷2 voltage divider, 8-sample average)

### Rendering Modes
| Mode | Time | Use |
|------|------|-----|
| PLANE_FALSE_DIFF | 200-400ms | Default — ghosting compensation |
| REFRESH_FULL | 2-5s | Every 16 partials, clears ghosting |
| REFRESH_FAST | 300-500ms | Quick text updates |
| 4-GRAY | 2-5s | Rich grayscale (2-bit, 4 levels) |

### Button Actions
| Duration | Action |
|----------|--------|
| Short (<1s) | **Force immediate refresh** (ECHO mod) |
| Medium (1-5s) | Double-click action |
| Long (5-15s) | WiFi credential reset |
| Extra-long (>15s) | Full factory reset |

### TurboQuant Pipeline
```
Mac M4 → Pillow render (800x480) →
  Floyd-Steinberg dither (4-gray) or threshold (1-bit) →
  Convert to grayscale PNG →
  Delta hash check (skip if unchanged) →
  curl POST to webhook image endpoint →
  ECHO renders on next wake cycle
```

Script: `shadow-lab/scripts/echo-turboquant.py`

## Firmware Mods (built, pending flash)

| Mod | File | Purpose |
|-----|------|---------|
| ECHO branding | WifiCaptive.h, mdns_discovery.cpp | Captive portal SSID + mDNS = "ECHO" |
| Self-hosted OTA | config.h, bl.cpp | Bypass BYOD OTA restriction via shadowlab.cc |
| Button refresh | bl.cpp | Short press = force immediate display update |

Binary: `.pio/build/xteink_x4/firmware.bin` (1,438 KB, 75.7% of bank)

## Optimization Opportunities

| Priority | Optimization | Impact |
|----------|-------------|--------|
| HIGH | Remove ESPAsyncWebServer | -200 KB flash |
| HIGH | Enable -Os + disable BLE | -230 KB flash |
| HIGH | HTTP connection reuse | +30-50% faster API |
| HIGH | WiFi modem sleep | +15-20% battery |
| MED | Fast partial waveform (defined, not loaded) | 300ms vs 2s refresh |
| MED | SSD1677 temperature read (stubbed at 22°C) | Better contrast |
| MED | Bufferless bb_epaper mode | Free 48 KB RAM |
| LOW | GPIO audit for deep sleep | Save 50-200 μA |

Full details: `xteink-firmware` skill → `references/optimization-plan.md`

## Architecture (SP-015)

```
ABSOLUTE:    ECHO e-paper (15 min) ← YOU ARE HERE
             shadowlab.cc (on-deploy)

RELATIVE:    shadow-tui (real-time)
             Chat sessions (per-message)

DUAL:        shadow-mcp (16 tools, on-demand)

GENERATIVE:  GLI (relative → promotes to absolute)
```

ECHO physically embodies SP-015: it IS the absolute addressing mode made tangible. E-ink chosen for persistence — updates persist until overwritten, no power to maintain. The shadow cast in permanent ink.

## Related Skills

| Skill | Scope |
|-------|-------|
| **hl-trmnl** | Headless CLI automation, all API operations |
| **trmnl-plugin-dev** | Plugin creation, webhook API, Liquid templates, 21 endpoints |
| **xteink-firmware** | ESP32-C3 builds, OTA, pin map, deep RE, optimization plan |

## Key URLs

| Resource | URL |
|----------|-----|
| Dashboard | https://trmnl.app/devices/41370/edit |
| Swagger | https://trmnl.com/api-docs/index.html |
| OpenAPI | https://trmnl.com/api-docs/openapi.yaml |
| Framework | https://trmnl.com/framework |
| Display page | https://shadowlab.cc/trmnl |
| Firmware source | `/Volumes/SHADOW/00-spaces/Personal/dev/clawd/workspaces/trmnl-x4/` |
