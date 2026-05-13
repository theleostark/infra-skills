---
name: SHADOW TUI
description: Use when launching, configuring, or extending the shadow-tui terminal runtime. Trigger on "shadow-tui", "TUI", "terminal dashboard", "Ink dashboard", "terminal runtime", "bun tui", "shadow dashboard terminal".
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-tui
  icon_style: craft-category-glyph-v1
---

# shadow-tui — SOTA Terminal Runtime

Ink (React for CLI) TUI at `/Volumes/SHADOW/shadow-lab/apps/shadow-tui/`.
Bun runtime (native JSX). Imports GLI modules directly.

## Launch

```bash
shadow-tui              # symlinked to ~/.local/bin/
# or
cd /Volumes/SHADOW/shadow-lab && bun apps/shadow-tui/src/app.js
```

Requires interactive TTY (won't work in piped/background mode).

## Panels

| Key | Panel | Data Source |
|-----|-------|-------------|
| a | All (default) | Everything |
| m | Mesh | probeAllNodes() from GLI mesh-dispatch |
| p | Persona | detectPersona() + getPersonaStats() |
| g | Gadgets | Static 8-gadget registry |
| t | Patents | Static 15-patent list |
| q | Quit | - |

## Architecture

- **Framework:** Ink v5 (React 18 for terminal)
- **Runtime:** Bun (native JSX transform, no babel)
- **Theme:** ShadowTech (violet/lavender/green/amber)
- **Data:** Imports directly from GLI packages via absolute paths
- **Surface mode:** RELATIVE (real-time, session-scoped)

## Extend

Add a new panel:
1. Create component function in `src/app.js`
2. Add keyboard binding in `useInput`
3. Add to render tree with tab filter


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

