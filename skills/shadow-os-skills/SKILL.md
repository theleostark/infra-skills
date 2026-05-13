---
name: SHADOW Os Skills
description: 'Use when the user needs: OS-level shadow ecosystem management — symlink graph, signal links, filesystem topology, launchd services, and mesh wiring. Use when managing links, checking OS-level shadow infrastructure, wiring new tools into the ecosystem, or auditing the filesystem graph.'
icon: icon.svg
triggers:
- shadow-link
- symlink
- signal link
- broken link
- wire up
- link audit
- launchd service
- os-level
- filesystem graph
- shadow-os
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-os-skills
  icon_style: craft-category-glyph-v1
---

# shadow-os-skills

OS-layer primitives for the shadow ecosystem. This is the filesystem/process/service substrate that everything else runs on.

## Core Tool: shadow-link

Location: `/Volumes/SHADOW/.shadow-a2a/shadow-link/shadow-link`
Linked to: `~/bin/shadow-link`

### Commands

| Command | Purpose |
|---------|---------|
| `shadow-link scan` | Discover all symlinks across ~/bin, SHADOW, ~/.Codex |
| `shadow-link audit` | Health check: broken links, stale signals, SHADOW-bound links |
| `shadow-link graph` | Visual category tree of the link ecosystem |
| `shadow-link create <name> <target> [--signal] [--category <cat>]` | Create managed link with optional signal emission |
| `shadow-link signal <name> [data]` | Emit manual signal to a2a system |
| `shadow-link repair` | Quarantine broken links to _broken-links/ |
| `shadow-link skills` | Map ~/.Codex/skills symlinks and sources |
| `shadow-link status` | Quick dashboard: link counts, FRIDAY a2a health |

### Signal Links

Signal links are symlinks that emit an a2a signal when created. This enables mesh-wide awareness of new tools.

```bash
# Create a signal-aware link
shadow-link create my-tool /path/to/target --signal --category shadow

# This does:
# 1. Creates ~/bin/_shadow/my-tool → /path/to/target
# 2. Creates ~/bin/my-tool → _shadow/my-tool (facade)
# 3. Emits signal to /Volumes/SHADOW/.shadow-a2a/signals/link-link-created.signal
# 4. Registers in shadow-link registry
```

## ~/bin Topology (Reverse Engineered)

The bin directory uses a two-layer pattern:

```
~/bin/
├── _dewey/          (26 tools) — Home automation, MCP gateway, wearables
├── _infra/          (15 tools) — Ports, volumes, SSH, monitoring
├── _agent-ops/      (9 tools)  — Policy checks, optimization, GLM
├── _repo-git/       (7 tools)  — Git sync, search, cleanup
├── _misc/           (6 tools)  — Misc utilities
├── _sysctl/         (5 tools)  — System control, agent maps
├── _openclaw/       (2 tools)  — OpenClaw search, drop
├── _broken-links/   (27 dead)  — Quarantined broken symlinks
├── shadow-link →    SHADOW/.shadow-a2a/shadow-link/shadow-link
├── shadow →         SHADOW/.shadow-a2a/shadow
├── shadow-a2a →     SHADOW/.shadow-a2a/shadow-a2a
├── shadow-beat →    SHADOW/.shadow-a2a/shadow-beat
└── mesh-msg →       SHADOW/.Codex-spaces/mesh/mesh-msg
```

**Pattern**: Category dirs (`_name/`) hold the real scripts. Facade links in ~/bin point to `_name/script`. SHADOW-bound tools link directly.

## Wiring a New Tool

When the user asks to "wire up" or "link" a new CLI:

1. Determine category (shadow, infra, agent-ops, misc, etc.)
2. Place script in the category dir or source location
3. Use `shadow-link create` with `--signal` if it should be mesh-aware
4. If it's SHADOW-bound, link directly: `~/bin/tool → /Volumes/SHADOW/path`

## launchd Services on FRIDAY

| Label | Port | Purpose |
|-------|------|---------|
| `cc.shadowlab.a2a-server` | 18790 | shadow-a2a HTTP server (always-on) |
| `io.shadow.mcp` | 3456 | Shadow MCP tunnel |
| `io.shadow.ambient` | — | Ambient intelligence daemon |
| `com.shadow.cairn.native` | — | Cairn native agent |
| `com.shadow.memory.native` | — | Memory service |
| `com.shadow-brain` | — | Shadow brain |

### Tailscale Serve Routes (FRIDAY)

```
https://friday-1.tail79a107.ts.net/
├── /            → http://127.0.0.1:18789  (OpenClaw gateway)
├── /.shadow     → http://localhost:18790   (shadow-a2a server)
└── /.well-known → http://localhost:18790   (agent discovery)
```

## Filesystem Signals

Signals live at `/Volumes/SHADOW/.shadow-a2a/signals/`. Any tool can emit a signal by writing a JSON file there. The a2a server on FRIDAY monitors these for mesh-wide events.

| Signal | Purpose |
|--------|---------|
| `beat.json` | Heartbeat from shadow-beat daemon |
| `mesh-state.json` | Current mesh node status |
| `jarvis-shadow-live.json` | JARVIS agent liveness |
| `link-*.signal` | Link creation/repair events from shadow-link |

## When to Use This Skill

- User says "link", "wire up", "symlink", "broken link"
- Auditing OS-level infrastructure
- Creating new CLI tools that need ecosystem integration
- Checking launchd services on mesh nodes
- Wiring signal-aware tools into the a2a mesh
