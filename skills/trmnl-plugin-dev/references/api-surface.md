# TRMNL Complete API Surface

> Reverse-engineered from OpenAPI spec at `https://trmnl.com/api-docs/openapi.yaml`
> + firmware source analysis + live endpoint probing (2026-03-26)

## Authentication

Two auth mechanisms:

| Method | Header | Format | Scope |
|--------|--------|--------|-------|
| **Device API Key** | `Access-Token` | Raw key (from /api/setup) | Device endpoints: /api/display, /api/current_screen, /api/log |
| **Account API Key** | `Authorization` | `Bearer user_xxxxx` | Account endpoints: /api/me, /api/devices, /api/playlists, /api/plugin_settings |

Get your Account API key from: https://trmnl.com/account

## Device API (Access-Token auth)

### GET /api/display — Fetch next screen (advances playlist)

**Headers:**
| Header | Required | Description |
|--------|----------|-------------|
| Access-Token | yes | Device API key |
| Battery-Voltage | no | e.g., 3.7 |
| Percent-Charged | no | e.g., 69.4 |
| FW-Version | no | e.g., 1.7.4 |
| RSSI | no | WiFi signal dBm, e.g., -69 |
| Height | no | Screen height pixels |
| Width | no | Screen width pixels |
| Special-Function | no | boolean |
| BASE64 | no | Request base64-encoded image |
| ID | no | Device MAC address |
| Model | no | Device model string |

**Response (200):**
```json
{
  "status": 200,
  "image_url": "https://trmnl.s3...",
  "filename": "plugin-xxxxx-timestamp",
  "refresh_rate": 903,
  "reset_firmware": false,
  "update_firmware": false,
  "firmware_url": "https://trmnl-fw.s3.../FW1.7.8.bin",
  "special_function": "none|identify|restart_playlist|send_to_me|guest_mode",
  "maximum_compatibility": true,
  "temperature_profile": "default|a|b|c",
  "action": null
}
```

### GET /api/display/current — Fetch current screen (NO playlist advance)

**Headers:** Access-Token only

**Response (200):**
```json
{
  "status": 200,
  "refresh_rate": 916,
  "image_url": "https://trmnl.s3...",
  "filename": "plugin-xxxxx",
  "rendered_at": "2023-01-01T00:00:00Z"
}
```

### POST /api/log — Submit device logs

**Headers:** Access-Token, ID
**Body:**
```json
{
  "logs": [
    {
      "timestamp": 1711454891,
      "codeline": 42,
      "sourceFile": "bl.cpp",
      "logMessage": "Display updated",
      "logId": 1234,
      "deviceStatusStamp": {
        "wifi_rssi_level": -45,
        "battery_voltage": 4.2,
        "refresh_rate": 900,
        "current_fw_version": "1.7.4",
        "free_heap_size": 120000
      }
    }
  ]
}
```

### GET /api/setup — Device registration

**Headers:** ID (MAC address), Model
**Response:** `{ status, api_key, friendly_id, image_url, message }`

## Account API (Bearer auth)

### GET /api/me — User profile

```json
{
  "data": {
    "id": 42,
    "name": "Jim Bob",
    "email": "jimbob@gmail.net",
    "first_name": "Jim",
    "last_name": "Bob",
    "locale": "en",
    "time_zone": "Eastern Time (US & Canada)",
    "time_zone_iana": "America/New_York",
    "utc_offset": -14400,
    "api_key": "user_xxxxxx"
  }
}
```

### GET /api/devices — List devices

```json
{
  "data": [{
    "id": 41370,
    "name": "My TRMNL",
    "friendly_id": "ABC-123",
    "mac_address": "12:34:56:78:9A:BC",
    "battery_voltage": 3.7,
    "rssi": -70,
    "sleep_mode_enabled": false,
    "sleep_start_time": 1320,
    "sleep_end_time": 480,
    "percent_charged": 85.0,
    "wifi_strength": 75.0
  }]
}
```

### GET /api/devices/{id} — Get device details
### PATCH /api/devices/{id} — Update device

Updatable fields: `sleep_mode_enabled`, `sleep_start_time`, `sleep_end_time`, `percent_charged`

### GET /api/playlists/items — List playlist items

Returns array of PlaylistItem with: device_id, plugin_setting, row_order, visible, rendered_at, mashup_id, mirror

### PATCH /api/playlists/items/{id} — Update playlist item

Body: `{ "visible": true/false }`

### GET /api/plugin_settings — List plugin settings

Query param: `?plugin_id=123` or `?plugin_id=calendars`

### POST /api/plugin_settings — Create plugin setting

Body: `{ "name": "My Plugin", "plugin_id": 123 }`

### DELETE /api/plugin_settings/{id} — Delete plugin setting

### GET /api/plugin_settings/{id}/data — Get plugin data (merge variables)
### POST /api/plugin_settings/{id}/data — Update plugin data (webhook push)

Body: `{ "merge_variables": { ... } }`

**UUID auth:** When using plugin UUID (not numeric ID), no bearer auth needed. This is the webhook endpoint.

### POST /api/plugin_settings/{id}/image — Upload webhook image

Content-Type: image/png, image/jpeg, image/bmp
Max: 5 MB, Rate: 12/hr

### GET /api/plugin_settings/{id}/archive — Download plugin archive
### POST /api/plugin_settings/{id}/archive — Upload plugin archive (multipart/form-data)

## Public API (No auth)

### POST /api/markup — Liquid template renderer

```bash
curl -X POST "https://trmnl.com/api/markup" \
  -H "Content-Type: application/json" \
  -d '{"markup": "Hello {{ name }}!", "variables": {"name": "World"}}'
# → {"data": "Hello World!"}
```

Accepts single string OR array of strings for batch rendering.

### GET /api/categories — List plugin categories

Returns: album, analytics, art, calendar, comics, crm, custom, discovery, ecommerce, education, email, entertainment, environment, finance, games, humor, images, kpi, life, marketing, music, news, notes, podcast, productivity, science, security, smart_home, social, sports, technology, tools, travel, utilities, weather

### GET /api/ips — Server IP addresses

Returns IPv4 and IPv6 addresses for allowlisting. Plugin polling only comes from these IPs.

### GET /api/models — All device models

Returns specs for: v2 (TRMNL X, 1872x1404), og_png (800x480 1-bit), og_plus (800x480 2-bit), kindle_*, tidbyt_64x32, and more.

### GET /api/palettes — Color palettes

Returns: bw (1-bit), gray-4 (2-bit), gray-16 (4-bit), gray-256 (8-bit), color-3bwr (B/W/Red), color-3bwy (B/W/Yellow), color-7 (7-color)

## MQTT (Sprite System — xteink-specific)

Default broker: `broker.hivemq.com:1883`
Topic: `xteink/x4/display`
States: idle, thinking, talking, excited, sleeping, error, alert, working

## Infrastructure

- **Server IP:** 157.230.59.36 (DigitalOcean)
- **Image storage:** AWS S3 (us-east-2), signed URLs (5-min expiry)
- **Firmware storage:** AWS S3 (trmnl-fw bucket)
- **TLS:** Let's Encrypt, TLSv1.3
- **HTTP/2:** Supported
- **Rails backend:** X-Runtime header, ETag caching
- **CDN:** None (direct to DO droplet)

## Undocumented Observations

1. **`/api/display` advances playlist** — each call shows the NEXT screen. Use `/api/display/current` to peek without advancing.
2. **BASE64 header** — pass `BASE64: true` to get the image base64-encoded in the response instead of a URL
3. **`image_url_timeout`** — server can override the default 15s image download timeout
4. **`maximum_compatibility`** — when true, device uses most compatible rendering mode
5. **Firmware URL always present** — even if `update_firmware` is false, the URL points to latest firmware
6. **Status 0 vs 200** — `/api/display` returns `"status": 0` (not 200) in the JSON body
7. **Webhook UUID = no auth** — POST to `/api/plugin_settings/{UUID}/data` works without bearer token
8. **Redirect following** — device firmware follows 301/302 redirects automatically
9. **Rate limit 429** — accepted silently by firmware, no error raised
10. **mDNS discovery** — device scans for `_http._tcp` services with "trmnl" in hostname for local BYOS servers
