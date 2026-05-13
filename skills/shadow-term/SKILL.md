---
name: SHADOW Term
description: Use when managing iTerm2 profiles as structured system prompts — colors, fonts, key bindings, triggers, badges, shell integration. Open split panes, configure profiles, map to ShadowSpaces identities. Base of shadow-term product. Trigger on "iTerm profile", "terminal config", "split pane", "shadow-term", "open terminal", "terminal theme", "shell config".
icon: icon.svg
metadata:
  category: enterprise/astemo
  family: shadow
  lifecycle: active
  canonical_slug: astemo-shadow-term
  icon_style: craft-category-glyph-v1
---

# shadow-term — iTerm2 Profile Engine

Manage iTerm2 profiles as configurable shell-first workspaces mapped to ShadowSpaces identities and mesh devices.

## Current Profile Model

Primary dynamic profile file:
- `~/Library/Application Support/iTerm2/DynamicProfiles/ShadowPair.json`

Primary local spaces on JARVIS:
- `shadow-unos` — dark local operator shell
- `shadow-duos` — light/review local shell
- `shadow-tres` — Astemo enterprise shell

Primary mesh workspaces:
- `JARVIS` — `sdluffy/JARVIS`
- `FRIDAY` — `leo/FRIDAY`
- `AURION` — `leo/AURION`
- `ULTRON` — `ishuru/ULTRON`

Rule:
- Treat these as shell-first environments that can run any CLI or TUI.
- Do not design them as single-purpose launchers unless the user explicitly asks for that.

## Split Pane Commands

```bash
# Vertical split in CURRENT session (default for monitors/tools)
osascript -e '
tell application "iTerm2"
    tell front window
        tell current session of current tab
            set newSession to (split vertically with default profile)
            tell newSession
                write text "COMMAND_HERE"
            end tell
        end tell
    end tell
end tell'

# Horizontal split (for logs/output below)
osascript -e '
tell application "iTerm2"
    tell front window
        tell current session of current tab
            set newSession to (split horizontally with default profile)
            tell newSession
                write text "COMMAND_HERE"
            end tell
        end tell
    end tell
end tell'
```

**Rule:** Prefer splitting in the current window when augmenting an existing workspace. Use a new window only when the user explicitly wants a standalone environment.

## Profile Configuration via AppleScript

```bash
# Set profile for current session
osascript -e '
tell application "iTerm2"
    tell current session of current tab of current window
        set profile name to "jarvis.dev.ishuru"
    end tell
end tell'

# Get current profile
osascript -e '
tell application "iTerm2"
    tell current session of current tab of current window
        return profile name
    end tell
end tell'
```

## Common Workflows

| Action | Command |
|--------|---------|
| Monitor (mactop) | Vertical split + `mactop` |
| SSH to mesh node | Vertical split + `ssh friday` / `ssh aurion` / `ssh ultron` |
| Serial monitor | Horizontal split + `pio device monitor` |
| Logs | Horizontal split + `tail -f /var/log/...` |
| Ollama chat | Vertical split + `ollama run qwen3:8b` |

## iTerm2 Configurable Elements (shadow-term mapping)

| Element | Profile Key | shadow-term Config |
|---------|------------|-------------------|
| Background color | `Background Color` | `theme.bg` |
| Foreground color | `Foreground Color` | `theme.fg` |
| Font | `Normal Font` | `theme.font` |
| Badge | `Badge Text` | `identity.badge` |
| Working directory | `Custom Directory` | `space.cwd` |
| Initial boot command | `Initial Text` | `space.bootstrap` |
| Command | `Command` | `space.shell` |
| Triggers | `Triggers` | `automation.triggers` |
| Key mappings | `Keyboard Map` | `keybindings` |

## Mesh Identity Mapping

| Device | User | SSH alias | Shell expectation |
|--------|------|-----------|-------------------|
| JARVIS | `sdluffy` | local | `zsh -l` local shell |
| FRIDAY | `leo` | `friday` | remote `zsh -l` |
| AURION | `leo` | `aurion` | remote `bash -l` |
| ULTRON | `ishuru` | `ultron` | remote `bash -l` |

## it2 Integration

The local `it2` helper is the preferred host-side automation layer when available.

Examples:

```bash
# List active sessions
it2 session list

# Open a new window with a specific profile
it2 window new --profile "JARVIS"

# Split current session vertically
it2 session split --vertical

# Send a command into the current session
it2 session send "ssh friday"
```

If `it2` is installed but cannot talk to iTerm2, enable:
- `iTerm2 -> Settings -> General -> Magic -> Enable Python API`

## Dynamic Profile Failure Triage

When iTerm profiles stop initializing correctly, do not assume the shell init is broken first. Validate the dynamic profile JSON files before debugging startup scripts.

Use this sequence:

```bash
python3 - <<'PY'
from pathlib import Path
import json
root = Path.home() / "Library/Application Support/iTerm2/DynamicProfiles"
for path in sorted(root.glob("*.json")):
    try:
        json.loads(path.read_text())
        print("OK", path.name)
    except Exception as exc:
        print("BAD", path.name, exc)
PY
```

Rules:

- If a profile file is invalid JSON, fix that first. iTerm can silently fail to load or refresh affected profiles.
- A common failure mode is a duplicated trailing object tail after the final `]}`. Trim the extra bytes and re-validate.
- Prefer Python `json.loads(...)` validation over `plutil -lint` for these dynamic profile JSON files on this host.
- After the JSON is valid, verify the profile `Initial Text` and `Set Local Environment Vars` still point at the live Shadow mount and init scripts.

Crystallized from:

- 2026-04-18 JARVIS dynamic profile repair
- Original task: debug why old terminal windows and new iTerm sessions were not initializing correctly
