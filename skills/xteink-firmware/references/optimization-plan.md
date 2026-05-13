# ECHO Firmware Optimization Plan

## Display Controller: SSD1677 (800x480)

### PLANE Modes
| Mode | Code | Effect |
|------|------|--------|
| PLANE_0 | 0 | Write plane 0 only (B/W) |
| PLANE_1 | 1 | Write plane 1 only (gray overlay) |
| PLANE_BOTH | 2 | Write both planes |
| PLANE_DUPLICATE | 3 | Copy plane 0 → 1 |
| PLANE_0_TO_1 | 4 | Send plane 0 data to plane 1 memory |
| PLANE_FALSE_DIFF | 5 | Force all pixels to update (xteink default) |

### 2-Bit (4-Gray) Rendering
- **Encoding:** 4 pixels per byte, bits [1:0]=px0, [3:2]=px1, [5:4]=px2, [7:6]=px3
- **Gray map:** Gray3=darkest, Gray0=lightest (inverted: `3 ^ (g >> 6)`)
- **Mode switch:** `bbep.setPanelType(dpList[iTempProfile].TwoBit)` then write PLANE_0 + PLANE_1
- **Dither:** 4x4 Bayer matrix in display.cpp (replaceable)

### Refresh Modes
| Mode | Time | Power | Ghosting |
|------|------|-------|----------|
| FULL | 2-5s | 3-5W | None |
| FAST | 300-500ms | 1-2W | Minimal |
| PARTIAL | 200-400ms | 0.5-1W | Medium |

### Temperature Bands
| Band | Range | Waveform Adjustment |
|------|-------|-------------------|
| COLD | <15°C | +20% phase duration |
| NORMAL | 15-30°C | Standard |
| WARM | 30-40°C | -10% duration |
| HOT | >40°C | Fastest, slight quality loss |

## Quick Wins (5 changes, ~300KB flash + 20% power savings)

### 1. Remove ESPAsyncWebServer (~200KB)
```ini
# platformio.ini — remove from [deps_app], add only to [env:TRMNL_X]
```

### 2. Compiler optimization -Os (~80-120KB)
```
# sdkconfig.xteink_x4_release
CONFIG_COMPILER_OPTIMIZATION_SIZE=y
```

### 3. Disable BLE (~150-200KB)
```
CONFIG_BT_ENABLED=n
```

### 4. HTTP connection reuse (+30-50% faster API)
```cpp
// lib/trmnl/include/http_client.h:55
https.setReuse(true);  // was false
```

### 5. WiFi modem sleep (+15-20% power)
```cpp
// After WiFi.begin() in bl.cpp
WiFi.setSleep(WIFI_PS_MAX_MODEM);
esp_wifi_set_max_tx_power(60);
```

## Medium Priority

| Optimization | Impact | File |
|-------------|--------|------|
| Dynamic CPU 80/160 MHz | 20-25% power | bl.cpp |
| Fast partial waveform (ALREADY DEFINED, not loaded) | 200-400ms faster | epd_optimizations.cpp |
| Sub-region update via `bbep.setAddrWindow()` | 50-200ms for status bar | display.cpp |
| Light sleep during download | 10-15% power | bl.cpp:947 |
| Replace String→char[] | 10-15KB heap | bl.cpp |
| DNS caching | 200-500ms faster | http_client.h |
| LTO in release | 5-10KB flash | platformio.ini |

## ANE/Mac Pre-Processing Pipeline

ESP32-C3 has no ML hardware. Offload to Mac M4:

```
Mac M4 ANE → Floyd-Steinberg dither → 4-gray quantize →
temporal delta → zlib compress → webhook push →
device renders delta region only
```

## Expected Results After Full Optimization

| Metric | Before | After |
|--------|--------|-------|
| Flash | 1,438 KB (75.7%) | ~900-1,000 KB (~50%) |
| RAM | 12.2% (40 KB) | ~8-10% |
| Boot | 8-10s | 6-8s |
| Battery | 2-4 weeks | 3-6 weeks |
| Refresh | 2-5s full | 300ms partial |

## Key Source Files

| File | Lines | Purpose |
|------|-------|---------|
| src/display.cpp | 2,047 | Display logic, refresh |
| src/epd_optimizations.cpp | — | Temp, refresh, power |
| src/bl.cpp | — | Business logic, WiFi, API |
| bb_ep.inl | — | Waveforms, LUTs, SPI commands |
| bb_ep_gfx.inl | — | Graphics primitives |
| arduino_io.inl | — | SPI I/O layer |
