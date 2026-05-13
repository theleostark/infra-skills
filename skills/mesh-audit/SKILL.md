---
name: Mesh Audit
description: Use when auditing the OpenClaw mesh network — disk usage, projects, services, models, connectivity, and health across all devices (ULTRON, JARVIS, FRIDAY, AURION). Use when the user wants to check system health, find storage issues, discover projects, or get a full mesh status report.
icon: icon.svg
user-invocable: true
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: alias
  canonical_slug: mesh-audit
  icon_style: craft-category-glyph-v1
  aliases:
  - /mesh-audit
  - mesh audit
  - mesh-audit
---

Run a comprehensive audit across the OpenClaw mesh network. Adapt scope based on what's requested.

## SSH Alias Reference

| Device | SSH Alias(es) | User | Notes |
|--------|---------------|------|-------|
| J.A.R.V.I.S | -- (local) | sdluffy | MacBook Air M4 |
| F.R.I.D.A.Y | `mm-m2-local` (LAN), `mm-m2` (Tailscale) | sdluffy | Mac Mini M2 |
| A.U.R.I.O.N | `starknet` | leo | Dell OptiPlex 9020 |
| U.L.T.R.O.N | `x-1` (Tailscale), `x-1-lan` (LAN) | ishuru | ThinkPad X1 Carbon |

## Audit Procedure

### 1. Connectivity Check
Test SSH to all 4 nodes. Use `ssh -o ConnectTimeout=5 -o BatchMode=yes <alias> echo ok` for each.
Report which are reachable before proceeding.

### 2. Per-Node Audit (run in parallel where possible)

For each reachable node, collect via SSH (or locally for JARVIS):

**Disk Overview:**
```bash
df -h / /data 2>/dev/null
```

**Top-Level Directory Sizes:**
```bash
du -sh ~/*/ 2>/dev/null | sort -rh | head -20
du -sh ~/.* 2>/dev/null | sort -rh | head -15
```

**Projects:**
```bash
du -sh ~/Projects/*/ ~/Work/*/ 2>/dev/null | sort -rh
```

**Services & Docker:**
```bash
docker ps --format "{{.Names}}: {{.Image}} ({{.Status}})" 2>/dev/null
# macOS:
launchctl list 2>/dev/null | grep -E 'sdluffy|cairn|shadowport|ollama'
# Linux:
systemctl --user list-units --type=service --state=running 2>/dev/null
```

**Ollama Models:**
```bash
ollama list 2>/dev/null
```

**Tailscale Status:**
```bash
tailscale status --json 2>/dev/null | head -50
```

**Uptime & Load:**
```bash
uptime
```

**Cleanable Space:**
- Cache dirs: ~/.cache, ~/.npm, ~/.gradle, ~/.m2
- Old AI tool caches: ~/.codex, ~/.gemini, ~/.amp, ~/.opencode
- Package manager caches: ~/.cargo, ~/.rustup, ~/.bun
- Stale exports/archives
- CoreSimulator volumes (JARVIS)

### 3. Cross-Node Analysis

After collecting per-node data, analyze:

- **Disk pressure**: Flag any node above 70% usage
- **Duplicate data**: Same projects/repos on multiple nodes
- **Orphaned data**: Old exports, stale caches, unused projects
- **Service health**: Down containers, failed services
- **Storage optimization**: What can be moved, archived, or deleted
- **Project distribution**: Which projects belong on which device based on role
- **Model efficiency**: Oversized models for available RAM (e.g., 70B on 8GB)
- **Tailscale peer status**: Any nodes offline or unreachable

### 4. Device Role Reference

| Device | Role | Should Have | Should NOT Have |
|--------|------|-------------|-----------------|
| ULTRON | Dev desktop (Omarchy) | Active dev projects, IDE configs, skills | Large archives, production services |
| JARVIS | Operator/admin (macOS) | OpenClaw codebase, project management, LLM models | Stale caches, old exports |
| FRIDAY | Gateway/agent (macOS) | Minimal footprint, gateway services, Cairn, small LLM models | Dev projects, build artifacts, models >8GB |
| AURION | Compute/storage (Ubuntu) | Docker services, archives, backups, ShadowLane | Desktop apps, user configs |

### 5. Update System Manifest

After collecting data, update `~/.macos-spaces/system-manifest.md`:
- Bump `generatedAt` timestamp
- Update free space numbers in Storage Topology sections
- Update service/container status in Services sections
- Update Tailscale peer online/offline status
- Add any newly discovered services, projects, or blockers
- Remove resolved blockers

### 6. Regenerate Dashboard

After manifest update, regenerate `/Users/sdluffy/Projects/system-manifest-dashboard.html`:
- Read the updated system-manifest.md
- Use the cc-html-page skill pattern and cc-design-system tokens
- Ensure all 6 tabs reflect current data: Fleet, Network, Storage, Services, Mobile, Blockers

### 7. Store Audit to Cairn

If Cairn is reachable (`http://100.76.188.80:8000/health`), store the audit summary:
```bash
curl -X POST http://100.76.188.80:8000/api/memories \
  -H "Content-Type: application/json" \
  -d '{"content": "<audit summary text>", "project": "mesh-audit", "metadata": {"type": "audit", "timestamp": "<ISO date>"}}'
```

### 8. Output Format

Present findings as:
1. **Health Summary** — traffic-light status per node (green/yellow/red)
2. **Storage Map** — where space is used, what's cleanable
3. **Service Status** — running/stopped/unhealthy per device
4. **Changes Detected** — what changed since last audit
5. **Action Items** — prioritized list of optimizations

## Skill Improvement Protocol

After each audit, update this skill file with:
- New directories or services discovered
- Changed device roles or storage layout
- Better audit commands for specific OS (macOS vs Linux differences)
- Performance optimizations (parallel SSH, faster scanning)
- New health check endpoints discovered

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

