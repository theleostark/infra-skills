# xteink_x4 Firmware Architecture (Deep RE)

## Boot Sequence
1. `checkIfAlreadyPassed()` — QA factory test check
2. `esp_ota_mark_app_valid_cancel_rollback()` — OTA rollback protection
3. `bl_init()` — Business logic init (WiFi, preferences, display)
4. `mdns_discovery_init()` — Local server scanning
5. `mqtt_sprite_init()` — MQTT sprite system
6. `sprite_renderer_init()` — Display sprite renderer

## Main Loop (cooperative, not preemptive)
1. `bl_process()` — WiFi → API → image download → render → sleep
2. `mqtt_sprite_loop()` — MQTT message check
3. `sprite_update_animation()` — Frame advance
4. `mdns_discovery_browse()` — mDNS scan every 60s

## Sleep/Wake
- **Deep sleep:** RTC timer + GPIO wakeup (GPIO 3 button)
- **MOSFET hold:** GPIO 13 keeps battery power during sleep
- **Default:** 900s (15 min)
- **RTC persists:** `need_to_refresh_display`, `iUpdateCount`

## Button Actions (GPIO 3, active LOW)
| Duration | Action |
|----------|--------|
| <1s | Short press (check double-click within 800ms) |
| 1-5s | Medium hold (double-click action) |
| 5-15s | Long press → WiFi credential reset |
| >15s | Soft reset → Full factory reset |

## WiFi
- **Captive portal SSID:** "TRMNL" (open, channel 6)
- **Max profiles:** 5 saved networks
- **Max SSID/password:** 40 chars each
- **Retry:** 60s → 180s → 300s → 900s (exponential backoff)
- **Enterprise:** WPA2-EAP supported

## Display Pipeline
```
HTTP Stream → malloc buffer → Format detect (PNG/JPEG/BMP) →
Decoder init → Line-by-line draw callback → bb_epaper SPI → Display
```
- **SPI clock:** 20 MHz
- **Temperature profiles:** DEFAULT, A, B, C (stored in NVS)
- **Write mode:** PLANE_FALSE_DIFF (ghosting compensation)
- **Max image:** 750KB (PSRAM), 90KB (no PSRAM)

## OTA
- **BYOD: NOT SUPPORTED** — must flash via USB from trmnl.com/flash
- Dual-bank A/B with rollback disabled by default
- No firmware signature verification

## MQTT Sprites
- **Broker:** broker.hivemq.com:1883 (configurable)
- **Topic:** xteink/x4/display
- **States:** idle, thinking, talking, excited, sleeping, error, alert, working
- **Sprite size:** 200x200px, max 4 frames, max 4096 bytes/frame

## mDNS / BYOS
- Scans `_http._tcp` for "trmnl" in hostname
- Caches local server URL in preferences
- Empty filename from API → activates local mode
- Switches API base from trmnl.app to local server

## NVS Keys
| Key | Purpose |
|-----|---------|
| `api_key` | Device API key |
| `api_url` | Custom API endpoint |
| `friendly_id` | Device name |
| `refresh_rate` | Sleep time (seconds) |
| `temp_profile` | Display temperature compensation |
| `filename` | Current image filename |
| `log_[0-9]` | Circular log buffer |
| `wifi_[0-4]_ssid/pswd` | WiFi credentials |
| `local_server` | Cached BYOS server URL |
| `testPassed` | QA factory test flag |

## Special Functions (from API)
- identify, sleep, add_wifi, restart_playlist, rewind, send_to_me, guest_mode

## Power
- **Battery ADC:** GPIO 0, 8 samples averaged, ×2 for voltage divider
- **States:** Active (~200mA), Refresh (~400mA), Light sleep (~1mA), Deep sleep (~50μA)

## Security
- TLS via WiFiClientSecure (no cert pinning)
- API auth: Access-Token header
- NVS encrypted at filesystem level
- No firmware signature verification

## Hidden Features
- **QA mode:** Triggered by "TRMNL_QA" WiFi network
- **Hardcoded WiFi:** Define `HARDCODED_WIFI` for dev
- **Serial debug:** 115200 baud, optional 2s wait
- **NVS debug:** `log_nvs_usage()` at init
