---
name: Tailscale Mesh
description: Use when managing the OpenClaw Tailscale mesh — ACL policy, SSH modes, device tagging, service exposure (Serve/Funnel), connectivity diagnostics, and agent autonomy levels. Use when configuring Tailscale, debugging mesh connectivity, managing device access, or setting up service exposure.
icon: icon.svg
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: tailscale-mesh
  icon_style: craft-category-glyph-v1
---

Manage Tailscale mesh networking for the OpenClaw network. Handles ACL policy, SSH access modes, device tagging, service exposure, and connectivity diagnostics.

## Network Topology

```
OpenClaw Tailnet: tail79a107
Account: sabarish.duvvuru@gmail.com

Devices:
  ULTRON  (x-1)        100.84.85.75   — Arch Linux, dev desktop
  JARVIS  (sd-mba-client) 100.89.81.28 — macOS, operator
  FRIDAY  (friday-mac)  100.76.188.80  — macOS, gateway/agent
  AURION  (starknet)    100.70.136.30  — Ubuntu, compute/services

LAN IPs (preferred, bypass Docker iptables):
  ULTRON: 192.168.86.119  JARVIS: 192.168.86.124
  AURION: 192.168.86.87   FRIDAY: friday-mac.local
```

## ACL Architecture — Agent Autonomy Modes

The ACL uses tags to define access tiers with human-in-the-loop controls:

### Tag Hierarchy
| Tag | Purpose | Assigned To | Access Level |
|-----|---------|-------------|--------------|
| `tag:control` | Admin/operator nodes | ULTRON, JARVIS | Full mesh access |
| `tag:services` | Service hosts (Docker, ShadowLane) | AURION | HTTP/S from members, full from admin |
| `tag:dev` | Development environments | All 4 nodes | SSH with check (human approval) |
| `tag:agent` | Autonomous AI agents | FRIDAY, AURION agents | Auto-accept SSH, scoped network |
| `tag:profile` | Personal/identity nodes | JARVIS | Scoped access |

### SSH Autonomy Modes
| Mode | SSH Action | Behavior | Use Case |
|------|-----------|----------|----------|
| **Full Autonomy** | `accept` | Auto-approve, no human check | Agent-to-agent, CI/CD, automation |
| **Human-in-Loop** | `check` | Browser approval required | Dev SSH, sensitive operations |
| **Supervised** | `check` + `checkPeriod: "1h"` | Re-auth every hour | Extended agent sessions |
| **Locked** | (no rule) | Denied by default | Untrusted/retired devices |

### Optimal ACL Policy Template
```json
{
    "tagOwners": {
        "tag:control":      ["autogroup:admin"],
        "tag:services":     ["autogroup:admin"],
        "tag:dev":          ["autogroup:admin"],
        "tag:agent":        ["autogroup:admin"],
        "tag:profile":      ["autogroup:admin"],
        "tag:openclaw-vps": ["autogroup:admin"]
    },
    "acls": [
        {
            "action": "accept",
            "src":    ["autogroup:member"],
            "dst":    ["autogroup:self:*"],
            "comment": "All personal devices can reach each other on all ports"
        },
        {
            "action": "accept",
            "src":    ["tag:agent"],
            "dst":    ["tag:services:*", "tag:dev:*", "tag:agent:*"],
            "comment": "Agents can reach services, dev nodes, and other agents"
        },
        {
            "action": "accept",
            "src":    ["autogroup:member"],
            "dst":    ["tag:services:443", "tag:services:80", "tag:services:8080"]
        },
        {
            "action": "accept",
            "src":    ["autogroup:admin"],
            "dst":    [
                "tag:control:*", "tag:services:*", "tag:dev:*",
                "tag:agent:*", "tag:profile:*", "tag:openclaw-vps:*"
            ]
        }
    ],
    "grants": [
        {
            "src": ["autogroup:admin"],
            "dst": ["tag:control", "tag:services", "tag:dev", "tag:agent", "tag:profile"],
            "ip":  ["*"]
        },
        {
            "src": ["tag:agent"],
            "dst": ["tag:services", "tag:dev"],
            "ip":  ["*"],
            "comment": "Agent nodes get IP-level access to services and dev"
        },
        {
            "src": ["autogroup:member", "tag:control"],
            "dst": ["autogroup:internet"],
            "ip":  ["*"]
        }
    ],
    "ssh": [
        {
            "action": "accept",
            "src":    ["tag:agent", "tag:control"],
            "dst":    ["tag:agent", "tag:services", "tag:dev"],
            "users":  ["root", "autogroup:nonroot"],
            "comment": "Full autonomy: agents and control nodes auto-approve SSH"
        },
        {
            "action": "check",
            "src":    ["autogroup:member"],
            "dst":    ["tag:dev", "tag:profile", "autogroup:self"],
            "users":  ["autogroup:nonroot"],
            "comment": "Human-in-loop: personal SSH requires browser check"
        },
        {
            "action": "accept",
            "src":    ["autogroup:admin"],
            "dst":    ["tag:control", "tag:services", "tag:dev", "tag:agent", "tag:profile"],
            "users":  ["root", "autogroup:nonroot"],
            "comment": "Admin gets full SSH everywhere"
        }
    ],
    "tests": [
        {
            "src": "sabarish.duvvuru@gmail.com",
            "accept": ["tag:services:443", "tag:services:80", "tag:control:22", "tag:dev:22"]
        },
        {
            "src": "tag:agent",
            "accept": ["tag:services:18700", "tag:services:5000", "tag:dev:22"]
        }
    ]
}
```

## Procedures

### 1. Connectivity Diagnostics
```bash
# Check Tailscale status
tailscale status

# Check packet filter rules (needs --operator set)
tailscale debug netmap 2>&1 | python3 -c "
import json, sys
data = json.load(sys.stdin)
pf = data.get('PacketFilter', [])
print(f'PacketFilter rules: {len(pf)}')
ssh = data.get('SSHPolicy', {})
print(f'SSH rules: {len(ssh.get(\"rules\", []))}')"

# Ping test (uses disco protocol, always works)
tailscale ping <hostname>

# TCP connectivity test
ssh <node> 'nc -zw3 <tailscale-ip> <port>'

# Check if Tailscale SSH intercepts port 22
# If RunSSH is true, tailscaled handles SSH on port 22 for Tailscale IPs
tailscale debug prefs 2>&1 | grep RunSSH
```

### 2. Device Tagging
```bash
# Tag a device (requires admin in Tailscale admin console)
# Navigate to: https://login.tailscale.com/admin/machines
# Click device → Edit ACL tags → Add tag

# Or via API (requires API key):
# curl -X POST "https://api.tailscale.com/api/v2/device/{deviceId}/tags" \
#   -u "tskey-api-xxx:" \
#   -d '{"tags":["tag:services"]}'

# Verify tags
tailscale status --json | python3 -c "
import json, sys
for k,v in json.load(sys.stdin).get('Peer',{}).items():
    tags = v.get('Tags', [])
    if tags:
        print(f'{v[\"HostName\"]}: {tags}')"
```

### 3. Expose Services via Tailscale Serve
```bash
# Expose ShadowLane (AURION :18700) to tailnet
tailscale serve --bg 18700

# Expose with custom port
tailscale serve --bg --http 8080 18700

# Expose MCP gateway
tailscale serve --bg --https 443 localhost:3000

# List active serves
tailscale serve status

# Identity headers available to backends:
# Tailscale-User-Login, Tailscale-User-Name, Tailscale-User-Profile-Pic
```

### 4. Expose to Internet via Funnel
```bash
# Expose a service publicly (ports 443, 8443, 10000 only)
tailscale funnel --bg 443

# Check funnel status
tailscale funnel status

# ACL must include funnel nodeAttr:
# "nodeAttrs": [{"target": ["tag:services"], "attr": ["funnel"]}]
```

### 5. ACL Update Workflow
```bash
# 1. Open ACL editor
xdg-open "https://login.tailscale.com/admin/acls/file"

# 2. Edit in JSON editor
# 3. Click "Preview changes" to see diff
# 4. Click "Save" to apply
# 5. Verify propagation:
tailscale debug force-netmap-update
tailscale debug netmap 2>&1 | python3 -c "
import json, sys
pf = json.load(sys.stdin).get('PacketFilter', [])
print(f'PacketFilter rules: {len(pf)}')"
```

## Integration Points

### ShadowLane (Model Router)
- AURION :18700 — expose via `tailscale serve` for encrypted mesh access
- All nodes can then use `https://starknet.tail79a107.ts.net:18700/v1/`
- Identity headers enable per-user model routing

### AiFi Gateway
- Run on tagged `tag:services` node
- Expose via Tailscale Serve for mesh access
- Use Funnel (port 443/8443/10000) for external webhook endpoints
- Identity headers provide zero-trust auth without API keys

### MCP Servers
- Expose MCP servers via `tailscale serve --https 443 localhost:<mcp-port>`
- Access from any mesh node via `https://<hostname>.tail79a107.ts.net/`
- Agent nodes (`tag:agent`) get auto-accept SSH for remote MCP execution

### OpenClaw Agent Mesh
- Tag agent containers/processes with `tag:agent`
- Agents auto-SSH between nodes (no human check)
- Control nodes (ULTRON, JARVIS) supervise via `tag:control`
- Graduated autonomy: new agents start as `tag:dev` (check mode),
  promote to `tag:agent` (accept mode) after validation

## Known Issues
- Docker iptables on Linux nodes blocks Tailscale IPs
  - Fix: `sudo iptables -I INPUT -i tailscale0 -j ACCEPT`
  - Persist: `sudo iptables-save | sudo tee /etc/iptables/iptables.rules`
  - Or use `tailscale serve` which bypasses iptables entirely
- Tailscale SSH (`RunSSH: true`) intercepts port 22 for Tailscale IPs
  - Regular `ssh user@tailscale-ip` triggers Tailscale SSH (check mode)
  - Use LAN IPs to bypass Tailscale SSH enforcement
- `tailscale serve` requires HTTPS certs enabled (MagicDNS)
- Funnel limited to ports 443, 8443, 10000

## Skill Improvement
After each use, note:
- New services discovered or exposed
- ACL changes and their effects
- Connectivity issues and resolutions
- New device tags assigned

$ARGUMENTS


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

