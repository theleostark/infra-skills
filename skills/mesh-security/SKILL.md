---
name: Mesh Security
description: Use when managing the OpenClaw mesh security monitoring system — live collector, security dashboard, hardening automation, and Tailscale API integration. Use when checking mesh security status, managing the monitor daemon, viewing threats, or performing security hardening.
icon: icon.svg
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: mesh-security
  icon_style: craft-category-glyph-v1
---

Manage the OpenClaw Mesh Security Monitor system. The monitor collects Tailscale mesh telemetry every 60 seconds, detects security threats, and serves a live dashboard.

## Components

| Component | Location |
|-----------|----------|
| Monitor CLI | `~/.local/bin/shadowskills-mesh-monitor` |
| Dashboard | `~/mesh-security-dashboard.html` |
| Systemd timer | `~/.config/systemd/user/mesh-monitor.{service,timer}` |
| Data file | `/tmp/mesh-monitor-data.json` |
| Logs | `~/.local/share/mesh-monitor/collector.log` |
| API key | `~/.config/tailscale/api-key` |

## Common Operations

### Check mesh security status
```bash
shadowskills-mesh-monitor status
# Or quick collect + status:
shadowskills-mesh-monitor collect && shadowskills-mesh-monitor status
```

### Manage the daemon
```bash
# Install/enable 60s timer
shadowskills-mesh-monitor install

# Check timer status
systemctl --user status mesh-monitor.timer

# View recent logs
tail -20 ~/.local/share/mesh-monitor/collector.log

# Disable
shadowskills-mesh-monitor uninstall
```

### Serve dashboard via Tailscale
```bash
# Start serving on :8090 (accessible from all mesh nodes)
shadowskills-mesh-monitor serve
# URL: https://x-1.tail79a107.ts.net:8090/

# Stop serving
shadowskills-mesh-monitor unserve
```

### Security hardening
```bash
shadowskills-mesh-monitor harden
# Checks: iptables rules, Docker isolation, key expiry, SSH audit, MFA
```

### Tailscale API
```bash
shadowskills-mesh-monitor api-setup      # Configure API key
shadowskills-mesh-monitor api-devices    # List all devices with tags
shadowskills-mesh-monitor api-acl        # View current ACL policy
shadowskills-mesh-monitor api-audit      # Recent device events
shadowskills-mesh-monitor api-tag <host> <tag>  # Tag a device
```

## Threat Detection

The collector detects:
- Key expiry < 30 days
- Offline nodes
- Unknown/unrecognized peers
- DERP-only connections (no direct path)
- ACL rejections in journal logs

Threat levels: `healthy` (green), `warning` (yellow), `critical` (red)

Desktop notifications via `shadowskills-notify` on threats.

## Data Schema

`/tmp/mesh-monitor-data.json`:
- `nodes{}` — per-node status, IPs, tags, key expiry, traffic
- `network{}` — UDP/IPv4/IPv6, DERP latency
- `metrics{}` — Prometheus counters (dns, magicsock, controlclient)
- `security{}` — threats, warnings, alerts, offline nodes, ACL rejections

$ARGUMENTS


## Pipeline

```
Intent → Scan mesh → Collect data → Analyze → Report findings
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Summary report | Normal scan |
| `full` | Complete dump | Deep analysis |
| `json` | Raw data | Programmatic processing |

## Artifact Routing

- Scan reports: `ShadowArchive/80-reports/mesh-*.md`
- Audit logs: `ShadowArchive/80-reports/`

## Prerequisites

- Tailscale mesh connected
- SSH access to target nodes

## Contract

- Read-only scanning by default
- Never execute remediation without operator approval
- Include timestamps with all findings

