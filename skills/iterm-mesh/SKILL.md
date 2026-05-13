---
name: iTerm Mesh
description: Use when managing iTerm2 profiles, mesh panes, split layouts, agent grids, zen themes, A2A discovery, or terminal embodiment. Trigger on "iterm", "terminal", "split pane", "mesh grid", "zen theme", "profile", "open terminal", "spawn agents", "iTerm config", "terminal setup".
icon: icon.svg
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: iterm-mesh
  icon_style: craft-category-glyph-v1
---

# iterm-mesh — Unified iTerm2 Mesh Management

Single entry point for all iTerm2 terminal mesh operations. Combines shadow-term, spaces-skills, mesh-grid, zen-adapt, A2A discovery, and GLI embodiment.

## Components

| Tool | Location | Purpose |
|------|----------|---------|
| `itp` | shadow-lab/packages/iterm-proxy | Space launcher (iTerm + Chrome) |
| `mesh-grid` | .Codex-spaces/mesh/mesh-grid | Agent grid spawner |
| `mesh-msg` | .Codex-spaces/mesh/mesh-msg | Inter-session messaging |
| `zen-adapt` | .Codex-spaces/mesh/zen-adapt | Ambient light theme engine |
| `gli embody` | shadow-lab/packages/gli | Self-embodiment into iTerm vars |

## Quick Reference

### Open Spaces
```bash
itp open shadow            # Shadow space (iTerm + Chrome)
itp open light             # Light space
itp pair                   # Shadow-pair split (both profiles)
itp ls                     # List all spaces, profiles
```

### Agent Grid
```bash
mesh-grid shadow light     # 2-agent split
mesh-grid shadow light astemo  # 3-pane layout
mesh-grid --group dev      # Pre-defined dev group
mesh-grid --group full     # All agents
mesh-grid --group review   # Review layout
```

### Messaging
```bash
mesh-msg send light "review this commit"
mesh-msg send shadow "build complete"
mesh-msg check             # Check inbox
mesh-msg history           # Recent messages
```

### Zen Themes
```bash
zen-adapt                  # Apply theme based on ambient light
zen-adapt --watch          # Auto-adapt every 30s
zen-adapt --profile dev    # Dev color scheme
zen-adapt --zone           # Show current light zone
```

### GLI Embodiment
```bash
gli embody --space shadow --agent gli --mode dao
gli self                   # Show current iTerm2 context
```

### A2A Discovery
```bash
shadow-a2a ls              # List all agents
shadow-a2a cards           # Show agent capability cards
shadow-a2a mailbox         # Check A2A mailboxes
```

## iTerm2 User Variables (12 canonical)

Set per-pane for scripts, hooks, and daemons:

1. `user.shadow_space` — shadow/light/astemo
2. `user.shadow_agent` — jarvis-shadow/light/astemo/gli
3. `user.shadow_role` — creation/review/enterprise/generative
4. `user.shadow_mode` — yin/yang/dao/zen
5. `user.shadow_status` — active/idle
6. `user.shadow_node` — jarvis/friday/aurion
7. `user.shadow_device` — mba-m4
8. `user.shadow_mesh_ip` — Tailscale IP
9. `user.shadow_kernel` — kernel hash
10. `user.shadow_ambient` — dark/dim/normal/bright/sun
11. `user.shadow_session` — session ID
12. `user.shadow_gli` — active/null (GLI embodiment)

Query: `osascript -l JavaScript -e 'Application("iTerm2").currentWindow().currentTab().currentSession().variable({named:"user.shadow_space"})'`

## Dynamic Profiles

Location: `~/Library/Application Support/iTerm2/DynamicProfiles/ShadowPair.json`

Profiles: leo, ishuru, sabarish, sabbu (local) + friday, ultron, aurion (SSH mesh)

Color scheme: ShadowEmber (#110a06 bg, #f4d9b6 fg, #ffb346 cursor)

## File Locations

```
~/.Codex/skills/shadow-term/     shadow-term skill
~/.Codex/skills/spaces-skills/   spaces-skills skill
/Volumes/SHADOW/.Codex-spaces/   space configs + mesh tools
/Volumes/SHADOW/.shadow-a2a/      A2A infrastructure
~/Library/.../DynamicProfiles/    iTerm2 profiles
~/.ssh/config                     mesh SSH aliases
```

## Crystallized Runtime Pattern

iTerm2 is not just a terminal launcher. Treat it as a **runtime state surface**:

- user variables = live context bus
- pane layout = work-mode embodiment
- badge/title = compressed state display
- self-inspection = refinement loop

For enterprise cowork systems, this means iTerm2 can reflect:
- active meeting/program
- active stakeholder lens
- mode (`brief`, `live`, `evidence`, `actions`, `output`)
- confidence / follow-up state

Use iTerm2 to inspect itself and refine the UI:
1. render current pane layout
2. inspect pane titles, badges, overflow, and variable state
3. compare against desired end-state clarity
4. adjust layout or state mapping
5. re-render


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

