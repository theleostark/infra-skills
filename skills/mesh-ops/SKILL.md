---
name: Mesh Ops
description: 'Use when the user needs: OpenClaw mesh fleet — dispatch, scan, audit, path resolve, security, topology, agents, caches, sensors. Use on "mesh", "AURION", "FRIDAY", "JARVIS", "ULTRON", "fleet", "dispatch", "mesh health", "agent status", "cache cleanup", "mesh security", or any cross-device operation.'
icon: icon.svg
metadata:
  depends_on: [astemo-program-context,shadow-secure,tailscale-mesh]
  category: enterprise/astemo
  family: mesh
  lifecycle: active
  canonical_slug: mesh-ops
  icon_style: craft-category-glyph-v1
---

# Mesh Operations

Fleet: JARVIS (MBA local), FRIDAY (RPi gateway), AURION (OCI ARM), ULTRON (Arch desktop).

| Op | How |
|----|-----|
| **Dispatch** | `ant.mesh` or `ssh <node> <cmd>` |
| **Scan** | Find resources, keys, configs across fleet |
| **Audit** | Disk, services, models, connectivity per node |
| **Path** | Translate ShadowArchive paths per OS/node |
| **Security** | Live collector, Tailscale API, hardening |
| **Topology** | Device metadata, IPs, service routing |
| **Agents** | Running processes, MCP servers, CC sessions |
| **Cache** | node_modules, .pio, __pycache__ cleanup |
| **Sensors** | `den.environment` (light, battery, thermal, WiFi), `tap.tabs` (browser) |
| **Astemo context (fleet)** | Canonical bundle: **`spaces/professional/sduvvuru-qx/`** on the ☯Duality checkout (see `sduvvuru-qx/ASTEMO-BUNDLE.md` § *Mesh*). Other nodes: `git pull` that repo path, then sync `~/.agents/skills/astemo-work-context` (and `astemo-program-context`) so agents cite the same registry + routing. |

## Pipeline

```
Input (trigger: mesh, fleet, dispatch, node name, mesh health)
  ↓
Discovery (ant.mesh or ssh-based probe → node list + status)
  ↓
Route to sub-operation:
  ├── Dispatch → run command on target node
  ├── Scan → inventory resources, keys, configs fleet-wide
  ├── Audit → disk, services, models, connectivity per node
  ├── Path → translate ShadowArchive paths per OS/node
  ├── Security → Tailscale API, exposure check, hardening
  ├── Topology → device metadata, IPs, service routing map
  ├── Agents → running processes, MCP servers, CC sessions
  ├── Cache → node_modules, .pio, __pycache__ cleanup
  └── Sensors → den.environment, tap.tabs
  ↓
Execute across fleet (parallel where safe)
  ↓
Artifact Generation
  ├── ShadowArchive/80-reports/mesh-audit-YYYY-MM-DD.md
  ├── ShadowArchive/80-reports/mesh-topology-YYYY-MM-DD.md
  └── ShadowArchive/80-reports/mesh-security-YYYY-MM-DD.md
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Fleet status card: each node's health, uptime, key services | `mesh`, `mesh health`, `fleet` |
| `dispatch <node> <cmd>` | Execute command on specific node and return output | `dispatch`, `run on aurion` |
| `scan` | Inventory resources, configs, and keys across fleet | `scan fleet`, `mesh scan` |
| `audit` | Full per-node audit: disk, services, models, connectivity | `mesh audit`, `fleet audit` |
| `topology` | Device metadata, IPs, service routing map | `mesh topology`, `topology` |
| `security` | Tailscale ACL check, exposure audit, hardening recommendations | `mesh security` |
| `agents` | Running processes, MCP servers, agent sessions per node | `agent status`, `mesh agents` |
| `cache` | Cache directory scan and cleanup across fleet | `mesh cache cleanup` |
| `sensors` | Live sensor data from `den.environment` and `tap.tabs` | `mesh sensors` |
| `path <path>` | Translate a path to the correct form for each node OS | `path resolve`, `mesh path` |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Fleet audit | `ShadowArchive/80-reports/mesh-audit-YYYY-MM-DD.md` | Full per-node health report |
| Topology map | `ShadowArchive/80-reports/mesh-topology-YYYY-MM-DD.md` | Device metadata, IPs, routing |
| Security report | `ShadowArchive/80-reports/mesh-security-YYYY-MM-DD.md` | Exposure audit, ACL status |
| Cache report | `ShadowArchive/80-reports/mesh-cache-YYYY-MM-DD.md` | Fleet-wide cache sizes |
| Dispatch log | `ShadowArchive/80-reports/mesh-dispatch-YYYY-MM-DD.log` | Command execution results |

## Fallback Chain

1. **Primary:** `ant.mesh` for fleet discovery + `ssh <node>` for execution
2. **ant.mesh unavailable:** Fall back to hardcoded fleet roster (JARVIS, FRIDAY, AURION, ULTRON) + direct SSH
3. **SSH to node fails:** Try Tailscale hostname first, then IP, then check if node is in Tailscale `status`
4. **Node unreachable:** Report offline; continue with reachable nodes; note in report
5. **Tailscale down:** Report Tailscale status; attempt direct LAN IP if known; otherwise report unreachable
6. **Last resort:** Local-only operations on JARVIS; explicitly state that fleet is degraded to single-node

## Prerequisites

- Tailscale mesh active and authenticated
- SSH keys deployed to fleet nodes (or Tailscale SSH enabled)
- `ant.mesh` MCP tool available (optional, falls back to direct SSH)
- `den.environment` for hardware sensors (optional, graceful skip)
- `tap.tabs` for browser context (optional, graceful skip)
- `ssh` available locally
- `tailscale` CLI for status/ACL checks (optional)
- Astemo context synced via `spaces/professional/sduvvuru-qx/` git pull

## Error Handling

| Failure | Recovery |
|---------|----------|
| SSH connection refused | Check Tailscale status; try alternate port; report offline |
| SSH timeout (30s) | Mark node as unreachable; continue fleet scan; note in report |
| Tailscale not running | Report; suggest `tailscale up`; attempt LAN fallback if IP known |
| `ant.mesh` API error | Fall back to hardcoded roster + direct SSH; note degraded mode |
| Node returns error on dispatch | Capture stderr; report in dispatch log; continue with next node |
| Permission denied on node | Report; do not attempt privilege escalation; suggest manual fix |
| Mixed OS path issues | Use `path` sub-operation to translate; never guess Windows/Linux paths |
| Disk full on remote node | Report; do not push files to that node; warn in audit |

## Contract

- **Read-first fleet operations.** Fleet-wide scans and audits are read-only. Dispatch commands that modify state require explicit operator approval.
- **No auto-restart.** mesh-ops does not restart services, kill processes, or modify configs on remote nodes without explicit approval per command.
- **No credential exposure.** Fleet reports must not contain SSH keys, Tailscale auth keys, API tokens, or secrets. Redact before writing artifacts.
- **No auto-delete on cache cleanup.** Cache reports show sizes and locations. Actual cleanup requires separate operator approval.
- **Volume integrity.** Never push files to a node with <5% free disk space.
- **Tailscale ACL respect.** Never attempt to bypass ACL restrictions. If access is denied, report and suggest ACL update.
- **Externalization rule.** All fleet reports go to `ShadowArchive/80-reports/`. Never leave the only fleet state in chat or `/tmp`.
- **Do not** modify Tailscale ACLs, SSH configs, or firewall rules from mesh-ops. Route to `tailscale-mesh` or `shadow-secure` skills.
- **Do not** deploy new services or containers from mesh-ops. Route to node-specific skills.
- **Astemo context sync** is informational (git pull status). mesh-ops does not force-push context bundles.

## Pipeline

```
Intent → Resolve mesh targets → Dispatch/check → Collect results → Report
```

1. Identify operation type (dispatch, scan, audit, health check)
2. Resolve target nodes from device catalog
3. Execute via SSH/Tailscale or agent message
4. Collect and format results
5. Report with verifiable evidence

## Modes

| Mode | Output | When |
|------|--------|------|
| `health` | Node health summary | "mesh health", "check fleet" |
| `dispatch` | Route task to optimal node | "run on aurion", "dispatch" |
| `audit` | Full fleet inventory | "mesh audit", "inventory" |
| `scan` | Security/service scan | "mesh scan" |

## Artifact Routing

- Health reports: `ShadowArchive/80-reports/mesh-health-YYYY-MM-DD.md`
- Audit inventories: `ShadowArchive/80-reports/mesh-audit-YYYY-MM-DD.md`

## Fallback Chain

1. SSH via Tailscale (direct)
2. Agent message (if agent session exists on target)
3. Skip node and report as unreachable

## Prerequisites

- Tailscale connected (`tailscale status`)
- SSH config with mesh nodes
- `system/controls/device-catalog.json` accessible

## Error Handling

- **Node unreachable**: Mark as offline, continue with reachable nodes
- **Auth failure**: Report and skip, do not retry with different credentials
- **Timeout**: 30s default per node, configurable

## Contract

- Never execute destructive commands on mesh nodes without operator confirmation
- Report unreachable nodes explicitly — do not silently skip
- Include timestamps with all health data
