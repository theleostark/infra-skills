---
name: Mesh Scanner
description: 'Use when the user needs: Scan the OpenClaw mesh network for resources, API keys, configs, projects, and services across all devices (FRIDAY, JARVIS, ULTRON, AURION). Use when exploring what exists across the fleet, finding scattered configs, or auditing the mesh state.'
icon: icon.svg
command: mesh-scan
user_invocable: true
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: mesh-scanner
  icon_style: craft-category-glyph-v1
---

# Mesh Scanner — Cross-Device Resource Discovery

## Mesh Topology
| Codename | SSH Target | OS | Role |
|---|---|---|---|
| F.R.I.D.A.Y | localhost | macOS M2 | Always-on gateway, Ollama host |
| J.A.R.V.I.S | jarvis-local | macOS M4 | Operator, primary dev |
| U.L.T.R.O.N | x-1 | Arch Linux | Dev desktop, Hyprland |
| A.U.R.I.O.N | starknet | Linux | Starknet compute |

## Scan Commands

### Find .env files across mesh
```bash
# Local
find /Users/leo -maxdepth 4 -name ".env" -o -name ".env.local" 2>/dev/null | grep -v node_modules

# JARVIS
ssh jarvis-local 'find ~ -maxdepth 4 -name ".env" -o -name ".env.local" 2>/dev/null | grep -v node_modules'

# ULTRON
ssh x-1 'find ~ -maxdepth 4 -name ".env" -o -name ".env.local" 2>/dev/null | grep -v node_modules'

# AURION
ssh starknet 'find ~ -maxdepth 4 -name ".env" -o -name ".env.local" 2>/dev/null | grep -v node_modules'
```

### Find projects across mesh
```bash
# Look for package.json, Cargo.toml, go.mod, pyproject.toml
for host in localhost jarvis-local x-1 starknet; do
  echo "=== $host ==="
  if [ "$host" = "localhost" ]; then
    find /Users/leo -maxdepth 3 \( -name "package.json" -o -name "Cargo.toml" -o -name "go.mod" -o -name "pyproject.toml" \) 2>/dev/null | grep -v node_modules
  else
    ssh $host 'find ~ -maxdepth 3 \( -name "package.json" -o -name "Cargo.toml" -o -name "go.mod" -o -name "pyproject.toml" \) 2>/dev/null | grep -v node_modules' 2>/dev/null
  fi
done
```

### Scan for API keys in env vars
```bash
for host in localhost jarvis-local x-1; do
  echo "=== $host ==="
  if [ "$host" = "localhost" ]; then
    env | grep -iE "(api|key|token|secret)" | grep -v "^_" | grep -v PATH
  else
    ssh $host 'env | grep -iE "(api|key|token|secret)" | grep -v "^_" | grep -v PATH' 2>/dev/null
  fi
done
```

### Check running services
```bash
# Local services
curl -s http://localhost:11434/api/tags | python3 -c "import sys,json; [print(m['name']) for m in json.load(sys.stdin).get('models',[])]" 2>/dev/null
lsof -iTCP -sTCP:LISTEN -P 2>/dev/null | grep -E ':(11434|18789|18790|18800|5555|5556)\s'

# JARVIS services
ssh jarvis-local 'lsof -iTCP -sTCP:LISTEN -P 2>/dev/null | head -20'
```

## Known Resource Locations
- **FRIDAY**: ~/mcp-gateway, ~/Projects/open-brain, Ollama models
- **JARVIS**: ~/openclaw, ~/Projects/comed, ~/Projects/claw-buddy, ~/shadowtech-site, ~/.Codex/skills
- **ULTRON**: ~/.config/openclaw
- **AURION**: Starknet contracts

## After Scanning
- Feed discovered API keys into ai-fi: `node src/ai-fi.mjs set KEY value`
- Update MEMORY.md with new discoveries
- Sync skills if new ones found: `scp -r` between peers


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

