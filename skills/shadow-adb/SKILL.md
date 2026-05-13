---
name: SHADOW ADB
description: Use when reading android phone screen via ADB — dump UI elements, extract text, get current app, take screenshots. Works with any connected Android device. Trigger on "what's on phone", "check phone", "phone screen", "ADB", "read phone", "phone UI".
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-adb
  icon_style: craft-category-glyph-v1
---

# shadow-adb — Android Phone Screen Reader

Read the current phone screen via ADB UI Automator.

## Prerequisites

- Phone connected via USB with USB debugging enabled
- `adb` in PATH

## Commands

```bash
# Check connection
adb devices -l

# What app is in foreground?
adb shell dumpsys window | grep "mCurrentFocus"

# Dump and read all screen text
adb shell uiautomator dump /sdcard/window_dump.xml
adb shell cat /sdcard/window_dump.xml | grep -o 'text="[^"]*"' | grep -v 'text=""'

# Get content descriptions (links, buttons)
adb shell cat /sdcard/window_dump.xml | grep -o 'content-desc="[^"]*"' | grep -v 'content-desc=""'

# Screenshot
adb shell screencap /sdcard/screen.png
adb pull /sdcard/screen.png /tmp/phone-screen.png

# Get current URL (from browser)
adb shell cat /sdcard/window_dump.xml | grep -o 'resource-id="[^"]*url[^"]*"[^/]*text="[^"]*"'
```

## One-Liner: Full Screen Read

```bash
adb shell uiautomator dump /sdcard/window_dump.xml 2>/dev/null && \
adb shell cat /sdcard/window_dump.xml | grep -o 'text="[^"]*"' | grep -v 'text=""' | sed 's/text="//;s/"$//'
```

## Phone Fleet

| Device | Model | ADB ID |
|--------|-------|--------|
| Pixel 10 Pro XL | mustang | 56221FDCQ0013G |
| Pixel 9 Pro | (GrapheneOS) | — |


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

