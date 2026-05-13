---
name: Mesh Context
description: 'Use when the user needs: Gather relevant project context from across the mesh network (JARVIS, FRIDAY, AURION, ULTRON).'
icon: icon.svg
user-invocable: true
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: mesh-context
  icon_style: craft-category-glyph-v1
---

# Mesh Context Gatherer

Search for and aggregate project-related artifacts, conversation history, memory files, and code across all reachable mesh devices.

## When to use

- Starting work on a project that may have prior history across devices
- Consolidating scattered knowledge about a topic
- Finding related code, configs, or docs across the fleet

## Procedure

### 1. Parse the query

Extract the search topic from `$ARGUMENTS`. This could be a project name, keyword, or concept (e.g., "confirmo", "desktop-pet", "openclaw gateway").

### 2. Check mesh reachability

Test connectivity to each peer in parallel. Use LAN aliases first (faster), fall back to Tailscale.

```bash
# LAN-first aliases (from ~/.ssh/config)
PEERS="friday-mac-local starknet x-1-lan"
for peer in $PEERS; do
  ssh -o ConnectTimeout=3 -o BatchMode=yes "$peer" echo "ok:$peer" 2>/dev/null &
done
wait
```

Device map:
- **friday-mac-local** / **mm-m2**: F.R.I.D.A.Y (Mac mini M2, always-on gateway)
- **starknet**: A.U.R.I.O.N (Ubuntu 24.04, dev anchor)
- **x-1-lan** / **x-1**: U.L.T.R.O.N (ThinkPad X1, Arch Linux)
- Local: J.A.R.V.I.S (this machine, MacBook Air M4)

### 3. Search local artifacts

Search the local machine first (fastest):

```bash
QUERY="<topic>"

# Project directories
find ~/Projects -maxdepth 2 -iname "*${QUERY}*" -type d 2>/dev/null

# Codex memory and sessions
grep -rl "$QUERY" ~/.Codex/projects/*/memory/ 2>/dev/null | head -20
grep -rl "$QUERY" ~/.Codex/memory/ 2>/dev/null | head -20

# Codex history
grep -rl "$QUERY" ~/.codex/ 2>/dev/null | head -20

# Gemini history
grep -rl "$QUERY" ~/.gemini/ 2>/dev/null | head -20

# OpenClaw workspace
grep -rl "$QUERY" ~/.openclaw/ 2>/dev/null | head -20

# System manifest
grep -i "$QUERY" ~/.macos-spaces/system-manifest.md 2>/dev/null

# Confirmo status and hooks
grep -rl "$QUERY" ~/.confirmo/ 2>/dev/null | head -20
```

### 4. Search remote peers

For each reachable peer, run parallel searches:

```bash
for peer in $REACHABLE_PEERS; do
  ssh -o ConnectTimeout=5 "$peer" bash -s <<'REMOTE' &
    QUERY="<topic>"
    echo "=== Projects ==="
    find ~/Projects -maxdepth 2 -iname "*${QUERY}*" -type d 2>/dev/null
    echo "=== Agent memory ==="
    grep -rl "$QUERY" ~/.Codex/projects/*/memory/ ~/.Codex/memory/ ~/.codex/ ~/.gemini/ 2>/dev/null | head -30
    echo "=== OpenClaw ==="
    grep -rl "$QUERY" ~/.openclaw/ 2>/dev/null | head -20
    echo "=== Config/Services ==="
    grep -rl "$QUERY" ~/.config/ 2>/dev/null | head -10
REMOTE
done
wait
```

### 5. Query Cairn knowledge graph (if reachable)

```bash
curl -s http://100.76.188.80:8000/api/memories/search \
  -H "Content-Type: application/json" \
  -d "{\"query\": \"$QUERY\", \"limit\": 10}" 2>/dev/null
```

### 6. Read and synthesize key files

For each discovered file, read the most relevant ones (memory files, AGENTS.md, README.md, package.json) to extract:
- What was built and why
- Key decisions made
- Unfinished work or next steps
- Related projects and dependencies

### 7. Present aggregated context

Output a structured summary:

```
## Mesh Context: <topic>

### Local (JARVIS)
- Projects: ...
- Memory: ...
- Sessions: ...

### FRIDAY (Mac mini)
- Projects: ...
- Services: ...

### AURION (Ubuntu)
- Projects: ...
- Docker: ...

### ULTRON (Arch)
- Projects: ...

### Cairn Knowledge Graph
- Related memories: ...

### Synthesis
- Key findings across mesh
- Recommendations for next steps
```

### 8. Optionally save to memory

If significant cross-device context was discovered, offer to save a summary to `~/.Codex/projects/<project>/memory/` for future sessions.

## Notes

- Never expose secrets or tokens found during search
- Prefer LAN aliases over Tailscale for speed
- If a peer is unreachable, note it and continue with available data
- Timeout SSH commands at 10s to avoid blocking
- Search is read-only — never modify files on remote peers

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

