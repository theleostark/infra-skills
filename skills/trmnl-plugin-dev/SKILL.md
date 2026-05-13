---
name: TRMNL Plugin Dev
description: 'Use when the user needs: Create, configure, and debug TRMNL e-paper private plugins — webhook data push, polling endpoints, Liquid/HTML templating, merge variables, image rendering, and playlist management. Use when building custom TRMNL plugins, pushing data to the e-paper display via API, writing TRMNL HTML templates, configuring webhook or polling strategies, working with merge variables, troubleshooting display rendering, or integrating any service with the TRMNL device. Trigger on "TRMNL pl'
icon: icon.svg
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: trmnl-plugin-dev
  icon_style: craft-category-glyph-v1
---

# TRMNL Plugin Development

Build custom plugins that push content to the TRMNL e-paper display. This skill covers the full plugin lifecycle: creating a private plugin, choosing a data strategy, writing templates, and pushing updates via webhook or polling.

The shadow-trmnl skill covers device configuration and the absolute surface concept. This skill is specifically about **building plugins** that generate and deliver content.

## Quick Reference

| Item | Value |
|------|-------|
| Device ID | 41370 |
| BYOD Token | stored credential; do not print in skill docs |
| Display | 800x480, 1-bit B/W or 2-bit (4-gray) |
| Developer URL | https://trmnl.com/devices/41370/developer/edit |
| Docs | https://docs.trmnl.com |
| Framework CSS | https://trmnl.com/framework |
| Rate limit | 12/hr standard, 30/hr TRMNL+ |
| Refresh interval | Currently 14 min (configurable 1-1440 min) |

## Plugin Types

| Type | Cost | Complexity | When to Use |
|------|------|-----------|-------------|
| **Recipes** | Free | Low | Built-in plugins, no external deps |
| **Private Plugins** | $20 one-time | Medium | Custom HTML + webhook/polling |
| **Marketplace** | Free to publish | High | Public plugins requiring OAuth2 |

For Shadow Lab work, **Private Plugins** are the right choice — full control, webhook push, custom markup.

## Data Strategies

Choose one per plugin. This determines how content gets to the device.

### Strategy 1: Webhook (Push)

You POST data to TRMNL when it changes. Best when you control when updates happen (e.g., a cron job, a deploy hook, a mesh event).

```bash
# Push merge variables to a private plugin
curl -X POST "https://trmnl.app/api/custom_plugins/{PLUGIN_UUID}" \
  -H "Content-Type: application/json" \
  -d '{
    "merge_variables": {
      "title": "Mesh Status",
      "nodes_online": 7,
      "last_anchor": "2026-03-26T10:31:00Z"
    }
  }'
```

The plugin UUID comes from the TRMNL dashboard when you create a private plugin.

**Rate limits:**
- Standard: 12 pushes/hour, max 2 KB per request
- TRMNL+: 30 pushes/hour, max 5 KB per request

**Merge strategies:**

```json
// Default: full replace
{ "merge_variables": { "title": "New" } }

// Deep merge: update nested keys without replacing siblings
{ "merge_variables": { "nested": { "key": "value" } }, "merge_strategy": "deep_merge" }

// Stream: append to arrays (great for logs, feeds)
{ "merge_variables": { "messages": ["new item"] }, "stream_limit": 50 }
```

**Read current state:**
```bash
curl "https://trmnl.app/api/custom_plugins/{PLUGIN_UUID}"
# Returns current merge_variables
```

### Strategy 2: Polling (Pull)

TRMNL fetches content from your URL(s) on each refresh cycle. Best when you have a web endpoint that always has fresh data (e.g., an API route on shadowlab.cc).

Set the polling URL in the TRMNL dashboard. Supports:
- JSON, RSS, XML, plaintext, CSV responses
- Multiple endpoints (line-separated, merged as `IDX_0`, `IDX_1`, etc.)
- Liquid interpolation in URLs for dynamic values

```
# Single endpoint
https://shadowlab.cc/api/trmnl/mesh-status

# Multiple endpoints (line-separated in TRMNL dashboard)
https://shadowlab.cc/api/trmnl/mesh
https://shadowlab.cc/api/trmnl/patents
https://shadowlab.cc/api/trmnl/anchor
```

Multiple endpoint responses merge into indexed variables: `{{ IDX_0.nodes }}`, `{{ IDX_1.count }}`, etc.

**Dynamic URLs with Liquid:**
```liquid
https://api.example.com/data?since={{ "now" | date: "%Y-%m-%d" }}
```

### Strategy 3: Webhook Image (Experimental)

Push a pre-rendered image directly — bypass HTML templating entirely.

```bash
curl -X POST "https://trmnl.app/api/custom_plugins/{PLUGIN_UUID}" \
  -H "Content-Type: image/png" \
  --data-binary @display.png
```

**Image requirements:**
- Exactly 800x480 pixels
- 1-bit or 2-bit color depth
- PNG (recommended), BMP3, or JPEG
- Max 5 MB

This is useful when rendering complex visualizations server-side (e.g., with Puppeteer or Cairo) rather than relying on TRMNL's HTML renderer.

## HTML Templating

TRMNL renders your HTML template with the TRMNL CSS framework and Liquid for variable interpolation. The rendered HTML is converted to a 1-bit or 2-bit image on TRMNL's servers.

### Layout Classes

```html
<!-- Full screen -->
<div class="full">
  <h1>{{ title }}</h1>
  <p>{{ description }}</p>
</div>

<!-- Half screen (side by side) -->
<div class="half-vertical">Left content</div>
<div class="half-vertical">Right content</div>

<!-- Quadrants -->
<div class="quadrant">Top-left</div>
<div class="quadrant">Top-right</div>
<div class="quadrant">Bottom-left</div>
<div class="quadrant">Bottom-right</div>
```

### Liquid Templating

Variables from merge_variables or polling responses are accessible via `{{ variable }}`:

```html
<div class="full">
  <h1>{{ title }}</h1>

  <!-- Iteration -->
  {% for node in nodes %}
    <span>{{ node.name }}: {{ node.status }}</span>
  {% endfor %}

  <!-- Conditionals -->
  {% if alert_count > 0 %}
    <p class="alert">{{ alert_count }} alerts</p>
  {% endif %}

  <!-- Filters -->
  <p>Updated: {{ updated_at | date: "%b %d, %H:%M" }}</p>
  <p>{{ description | truncate: 100 }}</p>
</div>
```

### Design for E-Paper

The display is 800x480, 1-bit (black/white) or 2-bit (4 grayscale levels). Design accordingly:

- **No color** — everything renders as black, white, or gray
- **High contrast** — thin lines and small text may not render well
- **No animation** — static image, refreshes every N minutes
- **Simple layouts** — the CSS framework handles responsive sizing
- **Test at https://trmnl.com/framework** — preview your markup before deploying

### Example: Shadow Lab Mesh Status Plugin

```html
<div class="full">
  <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 16px;">
    <h1 style="font-size: 24px; margin: 0;">Shadow Lab</h1>
    <span style="font-size: 14px; opacity: 0.6;">{{ updated_at }}</span>
  </div>

  <div style="display: flex; gap: 24px; margin-bottom: 16px;">
    <div>
      <span style="font-size: 36px; font-weight: bold;">{{ nodes_online }}</span>
      <span style="font-size: 14px;">/{{ nodes_total }} nodes</span>
    </div>
    <div>
      <span style="font-size: 36px; font-weight: bold;">{{ patent_count }}</span>
      <span style="font-size: 14px;">patents</span>
    </div>
    <div>
      <span style="font-size: 36px; font-weight: bold;">{{ skill_count }}</span>
      <span style="font-size: 14px;">skills</span>
    </div>
  </div>

  {% for node in nodes %}
  <div style="display: flex; justify-content: space-between; padding: 4px 0; border-bottom: 1px solid #ccc;">
    <span>{{ node.name }}</span>
    <span>{% if node.online %}&#9679;{% else %}&#9675;{% endif %}</span>
  </div>
  {% endfor %}

  <div style="margin-top: 12px; font-size: 12px; opacity: 0.5;">
    kernel: {{ kernel_hash | truncate: 12, "" }} | anchor: {{ last_anchor }}
  </div>
</div>
```

Corresponding webhook push:

```bash
curl -X POST "https://trmnl.app/api/custom_plugins/{UUID}" \
  -H "Content-Type: application/json" \
  -d '{
    "merge_variables": {
      "nodes_online": 7,
      "nodes_total": 8,
      "patent_count": 15,
      "skill_count": 97,
      "kernel_hash": "sha256:9951ab3f",
      "last_anchor": "10:31",
      "updated_at": "Mar 26, 10:31",
      "nodes": [
        {"name": "JARVIS", "online": true},
        {"name": "AURION", "online": true},
        {"name": "FRIDAY", "online": true},
        {"name": "ULTRON", "online": false},
        {"name": "SENTRY", "online": true}
      ]
    }
  }'
```

## Creating a Private Plugin

1. Go to https://trmnl.com/plugins (or Devices > Manage Plugins)
2. Click "Create Private Plugin" (requires $20 developer upgrade)
3. Choose data strategy: **Webhook** or **Polling**
4. If webhook: copy the plugin UUID for your POST endpoint
5. If polling: enter your endpoint URL(s)
6. Write your HTML markup in the template editor
7. Set the refresh interval
8. Add to your device playlist

## Playlist Management

- Each device has a playlist of plugins that auto-advance
- Default: each item shows for 7 days at device refresh interval
- Custom duration per item
- Drag/drop ordering in the TRMNL dashboard
- Time-of-day and day-of-week scheduling available

## Automation Patterns

### Cron-Based Push (via Shadow Lab API)

Add a cron endpoint that gathers mesh/kernel state and pushes to TRMNL:

```typescript
// apps/api — FastAPI route or apps/web — Astro API route
export async function pushToTrmnl(data: Record<string, any>) {
  const PLUGIN_UUID = process.env.TRMNL_PLUGIN_UUID;
  await fetch(`https://trmnl.app/api/custom_plugins/${PLUGIN_UUID}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ merge_variables: data }),
  });
}
```

### Deploy Hook Push

After deploying shadowlab.cc, push fresh data to the display:

```bash
# In deploy script or CI
curl -X POST "https://trmnl.app/api/custom_plugins/${TRMNL_PLUGIN_UUID}" \
  -H "Content-Type: application/json" \
  -d "$(node -e "
    const data = require('./apps/web/src/data/fleet.ts');
    console.log(JSON.stringify({ merge_variables: data }));
  ")"
```

### Server-Side Image Rendering

For complex visualizations, render the image server-side and push it:

```python
# Using Pillow to generate 800x480 1-bit image
from PIL import Image, ImageDraw, ImageFont
import requests

img = Image.new('1', (800, 480), 1)  # 1-bit, white background
draw = ImageDraw.Draw(img)
draw.text((20, 20), "Shadow Lab Mesh", fill=0)
# ... draw your visualization ...

buf = io.BytesIO()
img.save(buf, format='PNG')
buf.seek(0)

requests.post(
    f"https://trmnl.app/api/custom_plugins/{PLUGIN_UUID}",
    headers={'Content-Type': 'image/png'},
    data=buf.read()
)
```

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| 429 Too Many Requests | Rate limit exceeded | Wait 5 min, reduce push frequency |
| 422 Unprocessable Entity | Bad image format or size | Ensure exactly 800x480, 1-bit/2-bit |
| Display shows old content | Cached; hasn't refreshed yet | Wait for next refresh cycle |
| Liquid variables empty | merge_variables keys don't match template | Check spelling, use GET to verify current state |
| Template not rendering | CSS/HTML too complex | Simplify; test at trmnl.com/framework |

## Related Skills

- **shadow-trmnl** — Device configuration, absolute surface concept, kernel display
- **xteink-firmware** — ESP32-C3 firmware builds, OTA, pin map, flash management

## Hidden/Undocumented Endpoints

These were discovered via OpenAPI spec extraction and firmware RE:

### POST /api/markup — Free Liquid Template Renderer (PUBLIC, NO AUTH)

```bash
curl -X POST "https://trmnl.com/api/markup" \
  -H "Content-Type: application/json" \
  -d '{"markup": "Hello {{ name }}! Nodes: {{ count }}", "variables": {"name": "Shadow Lab", "count": 7}}'
# → {"data": "Hello Shadow Lab! Nodes: 7"}
```

Accepts single string OR array for batch rendering. Use this to test templates without deploying.

### GET /api/display/current — Peek without advancing playlist

Unlike `/api/display` which advances to the next item, this returns the current screen without side effects. Used by the Chrome extension for mirroring.

### BASE64 Header — Get image inline

Pass `BASE64: true` header to `/api/display` to receive the image base64-encoded in the response instead of an S3 URL.

### GET /api/models — All device specs (PUBLIC)

Returns display dimensions, bit depth, scale factors, palette IDs, image size limits for every device model (OG, X, Kindle, Tidbyt, BYOD).

### GET /api/palettes — Color palette options (PUBLIC)

Returns: bw (1-bit), gray-4 (2-bit), gray-16 (4-bit), gray-256 (8-bit), color-3bwr (B/W/Red), color-3bwy (B/W/Yellow), color-7 (7-color).

### Account API (Bearer auth with `user_xxxxx` key)

Full CRUD for devices, playlists, plugin settings. Get your key from https://trmnl.com/account

- `GET /api/me` — user profile, timezone, locale
- `GET /api/devices` — list all devices (battery, RSSI, sleep config)
- `PATCH /api/devices/{id}` — update sleep mode, charge percent
- `GET /api/playlists/items` — list playlist items
- `PATCH /api/playlists/items/{id}` — toggle visibility
- `GET /api/plugin_settings` — list all plugin instances
- `POST /api/plugin_settings` — create new plugin instance
- `DELETE /api/plugin_settings/{id}` — remove plugin
- `GET /api/plugin_settings/{id}/data` — read merge variables
- `POST /api/plugin_settings/{id}/data` — push merge variables (webhook)
- `POST /api/plugin_settings/{id}/image` — push webhook image
- `GET /api/plugin_settings/{id}/archive` — export plugin
- `POST /api/plugin_settings/{id}/archive` — import plugin

### Full API Reference

Read `references/api-surface.md` for the complete OpenAPI-sourced reference with all request/response schemas, infrastructure details, and firmware protocol analysis.

**Swagger UI:** https://trmnl.com/api-docs/index.html
**OpenAPI Spec:** https://trmnl.com/api-docs/openapi.yaml

## Resources

| Resource | URL |
|----------|-----|
| API Docs | https://docs.trmnl.com |
| Framework CSS | https://trmnl.com/framework |
| Plugin Help | https://help.trmnl.com/en/articles/9510536-private-plugins |
| Webhooks | https://docs.trmnl.com/go/private-plugins/webhooks |
| Templates | https://docs.trmnl.com/go/private-plugins/templates |
| Community Plugins | https://github.com/usetrmnl/plugins |
| BYOS Server | https://github.com/usetrmnl/byos_node_lite |
| Tricks & Tips | https://github.com/yunruse/trmnl-tricks |


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

