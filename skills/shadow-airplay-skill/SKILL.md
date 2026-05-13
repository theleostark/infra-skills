---
name: SHADOW Airplay Skill
description: Use when controlling apple TV and AirPlay devices from the terminal for mirroring, casting screenshots or videos, and discovering local AirPlay targets.
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-airplay-skill
  icon_style: craft-category-glyph-v1
---

# shadow-airplay-skill

Control and cast to Apple TV and AirPlay devices from the terminal. Use when the operator wants to "show on TV", "mirror to Apple TV", "cast to living room", or "show work on TV".

## When to Use
- Mirroring the current workspace to an Apple TV for review.
- Casting images (screenshots) or videos to an AirPlay target.
- Discovering device identifiers on the local network.

## Workflow

### 1. Discovery
Search for local AirPlay/atv devices to find IP and Identifier.
```bash
# List AirPlay targets (mDNS)
dns-sd -B _airplay._tcp
# Detailed pyatv scan (requires Python fix for 3.14+)
python3 -c "import asyncio; import pyatv; loop = asyncio.new_event_loop(); print(loop.run_until_complete(pyatv.scan(loop, timeout=5)))"
```

### 2. Mirroring (macOS Native)
The most reliable way to show an active terminal session or UI.
```applescript
tell application "System Events" to tell process "ControlCenter"
    set statusItems to every menu bar item of menu bar 1
    repeat with theItem in statusItems
        if description of theItem is "Control Center" then
            click theItem
            delay 1
            click (first group of window "Control Center" whose description is "Screen Mirroring")
            delay 1
            click (first checkbox of scroll area 1 of group 1 of window "Control Center" whose name contains "Living Room")
            exit repeat
        end if
    end repeat
end tell
```

### 3. Direct Casting (pyatv)
For pushing specific artifacts without mirroring the whole screen.
```bash
# Requires pyatv installed: pip install pyatv --break-system-packages --user
atvremote --name "Living Room" play_url "http://path/to/image.png"
```

## Key Decisions
- **Mirroring vs Casting**: Default to Mirroring for active work review (interactive). Use Casting for static artifacts or media.
- **Python 3.14+ Compatibility**: Always provide a manual loop to `pyatv.scan` as the default `asyncio.get_event_loop()` fails in 3.14+ without an active loop.

## Gotchas
- **Control Center descriptions**: UI elements in Control Center often lack names and must be targeted via `description`.
- **System Events permissions**: Requires Terminal/OpenCode to have Accessibility permissions.

## Crystallized From
- Session: 2026-04-19
- Original task: Configure Claude Code for z.ai and show work on Apple TV.
- Pattern extracted: Bridging macOS Control Center UI to AirPlay targets via AppleScript + pyatv discovery.


## Pipeline

```
Intent → Resolve target → Execute operation → Verify result → Report
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Standard output | Normal use |
| `verbose` | Detailed diagnostics | Debugging |
| `json` | Machine-readable output | Programmatic use |

## Artifact Routing

- Reports: `ShadowArchive/80-reports/`
- Config changes: `system/controls/` or `config/system/`

## Prerequisites

- SHADOW infrastructure accessible
- Appropriate auth/SSH for mesh targets

## Contract

- Never modify production config without operator confirmation
- Report all changes with before/after diff
- Preserve existing configurations with backup

