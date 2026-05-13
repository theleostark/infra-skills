---
name: SHADOW Relay Agent
description: 'Use when the user needs: Async A2A relay agent for live context delegation, VPN-bound operations, shared queues, browser fallback, and session handoff.'
icon: icon.svg
metadata:
  category: enterprise/fh4s
  family: shadow
  lifecycle: active
  canonical_slug: fh4s-shadow-relay-agent
  icon_style: craft-category-glyph-v1
---

# shadow-relay-agent

Async A2A relay agent for live context delegation between operator and agents.

## When to Use

Use this skill when:
- Delegating VPN-bound operations (M365, GitHub OAuth)
- Crossing async boundaries during long-running tasks
- Need agent continuation with fresh context
- Automating calendar/email/web scraping operations

## What It Does

- Loads live context from Dropbox FH4S, Smartsheet, Outlook calendar
- Checks data freshness (hourly/daily thresholds)
- Queues tasks for agents via Reminders app shared queue
- Handles 3-tier fallback for Chrome automation
- Manages session handoff at context limits

## Operations

### Get Live Context
```
relay-agent context
```
Returns current context state with fresh/stale sources and action items.

### Queue Agent Task
```
relay-agent queue --priority=P0 --description="Complete M25" --context="JP provided part numbers"
```
Queues async task for agent processing via Reminders app.

### Check Async Boundary
```
relay-agent check-boundary --operation=outlook
```
Returns whether operation requires async boundary crossing.

## Integration Points

- **Reminders app:** "Shadow Relay Queue" list in iCloud
- **Dropbox sync:** `ShadowArchive/20-ingested/dropbox/fh4s-ipm/a2a-config.json`
- **Smartsheet API:** Token from macOS Keychain
- **Chrome profiles:** Work (sduvvuru), Dev (ishuru), Personal (sdluffy)

## Fallback Protocol

When Chrome automation blocked:
1. Chrome DevTools MCP (primary)
2. Playwright CDP (OAuth/CAPTCHA blocks)
3. Vision extraction (last resort)

## Session Handoff

Auto-triggers at 90% context capacity:
- Extract open action items and pending decisions
- Generate handoff summary to `memory/session-handoffs/`
- Update MEMORY.md with handoff pointer
- Continue in new session with preserved context

## Examples

Delegate M365 operation:
```
relay-agent queue --priority=P0 --description="Upload meeting notes to SharePoint" --context="FH4S IPM meeting #20"
```

Check calendar freshness:
```
relay-agent context | grep -A 5 "outlook_calendar"
```

Async boundary check:
```
relay-agent check-boundary --operation=sharepoint
```

## Security Model

**Zero PII in Dropbox**: The a2a-config.json contains NO personally identifiable information, device fingerprints, or account details. All PII lives in local-only config: `ShadowArchive/90-sensitive-sealed/local-operator-config.json`.

**Portable vs Local**:
- **Portable** (Dropbox): Role, site, company, BU → context references only
- **Local** (sealed): Name, email, account mappings, credentials → never synced

**PII Enrichment**: Relay agent enriches context locally from sealed config before any display or queuing.

**Platform Agnostic**: No hardcoded paths or device-specific mount points. Uses environment variables and relative paths.

## Config Location

Portable config in Dropbox: `ShadowArchive/20-ingested/dropbox/fh4s-ipm/a2a-config.json`

Can be edited to adjust:
- Freshness thresholds (hourly/daily/weekly)
- Async boundary timeouts
- Priority filters
- Live context sources
- No device-specific paths or account details
