---
name: Launchd Mise Service
description: Use when running a mise-managed runtime (Erlang, Node, Ruby, Python) as a macOS launchd service, or when a launchd service crashes because it can't find the runtime binary or escript fails with erlexec not found.
icon: icon.svg
metadata:
  category: workflow/process
  family: workflow
  lifecycle: active
  canonical_slug: launchd-mise-service
  icon_style: craft-category-glyph-v1
---

# launchd + mise-managed Runtime Services

## Overview

macOS launchd does not inherit shell environment (no PATH, no shell profile). mise-managed runtimes need `mise exec --` as the launcher, with `WorkingDirectory` pointing to where `.mise.toml` lives.

## Critical Rules

1. **Never symlink runtime binaries** (e.g. `erl`, `escript`) to `~/.local/bin`. Erlang scripts have hardcoded `ROOTDIR` paths — the symlink breaks them.
2. **Always use `mise exec --`** as the ProgramArguments prefix — it injects the correct runtime PATH.
3. **Set `WorkingDirectory`** to the project dir containing `.mise.toml` — otherwise mise can't find the tool versions.
4. **Declare all env vars explicitly** in the plist `EnvironmentVariables` dict — launchd inherits nothing from the shell.

## Plist Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.example.service</string>

  <key>ProgramArguments</key>
  <array>
    <string>/Users/sdluffy/.local/bin/mise</string>
    <string>exec</string>
    <string>--</string>
    <string>/path/to/your/bin/service-binary</string>
    <string>--any-flags-here</string>
    <string>/path/to/config</string>
  </array>

  <!-- Required: mise looks for .mise.toml here to activate tool versions -->
  <key>WorkingDirectory</key>
  <string>/path/to/project/with/mise-toml</string>

  <key>EnvironmentVariables</key>
  <dict>
    <!-- Declare all env vars your service needs -->
    <key>HOME</key>
    <string>/Users/sdluffy</string>
    <key>MISE_DATA_DIR</key>
    <string>/Users/sdluffy/.local/share/mise</string>
    <!-- Add service-specific vars -->
    <key>MY_API_KEY</key>
    <string>value</string>
  </dict>

  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>

  <key>StandardOutPath</key>
  <string>/path/to/logs/stdout.log</string>
  <key>StandardErrorPath</key>
  <string>/path/to/logs/stderr.log</string>
</dict>
</plist>
```

## Workflow

```bash
# Validate plist syntax before loading
plutil -lint ~/Library/LaunchAgents/com.example.service.plist

# Create log directory if referenced in plist
mkdir -p /path/to/logs

# Load (auto-starts due to RunAtLoad)
launchctl load ~/Library/LaunchAgents/com.example.service.plist

# Check status: columns are PID, last-exit-code, label
# PID non-zero = currently running
launchctl list | grep example

# Reload after editing plist
launchctl unload ~/Library/LaunchAgents/com.example.service.plist
# edit plist...
launchctl load ~/Library/LaunchAgents/com.example.service.plist

# Check logs
tail -f /path/to/logs/stderr.log
```

## Diagnosing Crashes

`launchctl list` column 2 is the exit code of the **last** run. PID `-` or `0` = not running.

| Exit code | Meaning |
|---|---|
| `0` | Exited cleanly (but `KeepAlive` will restart) |
| `1` | Error — check stderr.log |
| `126` | Permission denied or binary not executable |
| `127` | Binary not found |

## Erlang/OTP Specific

For Erlang escript binaries (like Symphony):

```bash
# ✅ Correct: mise exec resolves the escript runtime
mise exec -- ./bin/my-escript-binary

# ❌ Wrong: direct call fails if erl is not in PATH
./bin/my-escript-binary

# ❌ Wrong: symlinking erl breaks because Erlang uses ROOTDIR-relative paths
ln -s ~/.local/share/mise/installs/erlang/28.4.1/bin/erl ~/.local/bin/erl
```

The escript shebang is `#!/usr/bin/env escript`. The `escript` binary then calls `erl`, which looks for `erlexec` relative to its own installation root. Symlinks break this chain.

## Common Mistakes

| Mistake | Fix |
|---|---|
| No `WorkingDirectory` | mise can't find `.mise.toml`, uses wrong runtime |
| Symlinked `erl`/`escript` | Remove symlinks, use `mise exec --` |
| No `HOME` in env vars | Many tools fail silently without `HOME` |
| Editing plist while service is running | Unload first, edit, reload |
| Secrets in plist committed to git | Keep plist in `~/Library/LaunchAgents/` (not repo), source from `~/.config/` |


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

