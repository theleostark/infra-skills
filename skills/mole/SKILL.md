---
name: 'Disk and RAM Pressure Response'
description: 'Use when the user needs: Disk and RAM pressure response playbook for JARVIS, covering quick diagnostics, safe reclaim actions, and the existing Mole agent lifecycle.'
icon: icon.svg
metadata:
  category: system/ops
  family: design
  lifecycle: active
  canonical_slug: mole
  icon_style: craft-category-glyph-v1
---

# Mole — Pressure Agent

Disk + RAM pressure management for JARVIS (16GB, 250GB).

## When to Trigger

- "memory", "RAM", "disk space", "pressure", "oom", "slow", "mole"
- `vm_stat` shows free pages < 5000
- `memory_pressure` warns nominal→warning
- `df -h` shows < 10GB free

## RAM Health (check first)

```bash
vm_stat | awk '/Pages free/{f=$3} /Pages active/{a=$3} /Pages inactive/{i=$3} END{pg=16384; printf "Active: %.1fGB\nInactive: %.1fGB\nFree: %.0fMB\n", a*pg/1073741824, i*pg/1073741824, f*pg/1048576}'
ps aux -m | awk 'NR==1 || $4>1.0'
```

**JARVIS baseline (idle):** ~6GB used. Alert at >10GB used.

### Top consumers (typical)
- opencode sessions: 800MB-1.5GB each
- Chrome: 400-600MB per tab/renderer
- iTerm2: 400MB
- PowerPoint: 280MB

### Quick RAM kills
- Orphan Chrome DevTools: `pkill -f "chrome-devtools-mcp/chrome-profile"`
- Idle opencode: `kill <pid>` (check `ps aux | grep opencode`)
- PowerPoint: `killall "Microsoft PowerPoint"`

## Disk Health

```bash
df -h /Volumes/☯Duality
du -sh ~/Library/Caches/Homebrew ~/Library/Caches/pip ~/.npm/_npx 2>/dev/null
```

### Quick disk reclaims
- `brew cleanup --prune=7d`
- `npm cache clean --force`
- `trash -e`

## Rules

- Never kill the active opencode session
- Never kill iTerm2
- Never sudo
- Report before acting: "X using YMB, kill? Y/n"
- Keep responses under 5 lines

## Existing Agent

LaunchAgent: `com.shadow.mole-agent`
Wrapper: `/opt/homebrew/bin/shadow-mole-agent`
Source: `~/.config/shadow/mole-agent/mole-agent.py`
State: `/tmp/shadow-mole-agent-state.json`
Log: `/tmp/shadow-mole-agent.log`
Interval: 300s (5 min)
Check: `launchctl list com.shadow.mole-agent`

### Known bugs fixed (2026-04-16)
- `tool_installer` → `tool_mole_installer` (NameError crash)
- `dry_run: True` → `False` (was observation-only for 11hrs)
- Plist now uses `/opt/homebrew/bin/shadow-mole-agent` wrapper (shows agent name in Activity Monitor)

## Crystallized From

- Session: 2026-04-16, JARVIS
- Task: Fix broken mole-agent + crystallize macOS-as-API patterns
- Pattern: LaunchAgent wrappers, RAM vs disk separation, vision model routing
