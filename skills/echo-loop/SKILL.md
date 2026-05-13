---
name: Echo Loop
description: 'Use when the user needs: ECHO device monitoring loop — check status, push frames, verify connectivity. Trigger on: echo loop, echo status loop, monitor echo, echo check, watch echo, echo heartbeat.'
icon: icon.svg
metadata:
  category: workflow/process
  family: workflow
  lifecycle: active
  canonical_slug: echo-loop
  icon_style: craft-category-glyph-v1
---

# echo-loop — ECHO Device Monitor

Periodic check of ECHO device health + optional frame push.

## Credentials

| Purpose | Value |
|---------|-------|
| Device Access Token | `1qsLJdoKJPtcyUTdTeqayg` |
| Device ID Header | `937793dbb2e716a7f8f273cde78ceb28` |
| Account API Key | `user_crfhaacrc798tqf80km1n9du` |
| Webhook UUID | `f5bc3d4e-ab85-495a-9f02-7d7c126b24ca` |

## Check Sequence

### 1. Device Display Status
```bash
curl -s "https://trmnl.app/api/display/current" \
  -H "Access-Token: 1qsLJdoKJPtcyUTdTeqayg" \
  -H "ID: 937793dbb2e716a7f8f273cde78ceb28"
```
Parse: status, refresh_rate, image_url, filename

### 2. Account Health
```bash
curl -s "https://trmnl.com/api/me" \
  -H "Authorization: Bearer user_crfhaacrc798tqf80km1n9du"
```

### 3. Report Format
```
ECHO [HH:MM] status=OK refresh=919s image=webhook-image-xxx.png
```

If status is not 200 or image is stale (>30 min), flag as WARNING.

## Arguments

| Arg | Action |
|-----|--------|
| (none) | Quick status check |
| `push` | Render + push new frame via TurboQuant |
| `full` | Status + playlist + device details |
