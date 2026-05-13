---
name: SHADOW MCP Gadgets
description: Use when interacting with shadow-mcp tools — arc (memory), esp (context), ant (mesh), tap (browser), owl (ambient), orb (voice), ink (writing), den (physical), factory (patents). Trigger on "shadow-mcp", "gadget", "arc.store", "esp.assemble", "ant.mesh", "tap.tabs", "owl.brief", "orb.say", "den.environment", "MCP tools", "16 tools".
icon: icon.svg
metadata:
  depends_on: [bee]
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-mcp-gadgets
  icon_style: craft-category-glyph-v1
---

# shadow-mcp — 8 Gadgets, 16 Tools

Federation MCP server at `/Volumes/SHADOW/shadow-lab/apps/shadow-mcp/`.
Registered in `/Volumes/SHADOW/.mcp.json`. Runs via `bun`.

## Quick Reference

| Gadget | Tools | What |
|--------|-------|------|
| ARC | arc.store, arc.search | Memory — store + semantic search |
| ESP | esp.assemble | Context — unified context assembly (replaces /renew-ctx) |
| ANT | ant.mesh | Mesh — fleet health via live Tailscale |
| TAP | tap.tabs, tap.execute | Browser — Chrome tabs + JS execution |
| OWL | owl.brief, owl.extract | Ambient — Bee AI conversations, commitments, facts |
| ORB | orb.say, orb.voices | Voice — macOS TTS |
| INK | ink.recognize, ink.notes | Writing — Phase 2 (iPad/Vision) |
| DEN | den.environment | Physical — brightness, battery, thermal, WiFi, uptime |
| factory | .new/.research/.draft/.list/.status | Patents — draft generation pipeline |

## Run Locally

```bash
# stdio mode (Codex MCP)
bun run /Volumes/SHADOW/shadow-lab/apps/shadow-mcp/src/index.ts

# local mode (all mocks, no external deps)
bun run /Volumes/SHADOW/shadow-lab/apps/shadow-mcp/src/index.ts --local
```

## Test a Tool

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}
{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"den.environment","arguments":{}}}' | bun run /Volumes/SHADOW/shadow-lab/apps/shadow-mcp/src/index.ts
```

## Add a New Tool

1. Create gadget in `src/gadgets/<name>/index.ts`
2. Add tool definition + handler in `src/gadgets/registry.ts`
3. Type-check: `npx tsc --noEmit`

## Pipeline

```
Input (trigger: shadow-mcp, gadget, arc.store, esp.assemble, ant.mesh, etc.)
  ↓
Route to Gadget:
  ├── ARC → memory store/search
  ├── ESP → context assembly
  ├── ANT → mesh fleet health
  ├── TAP → browser tabs/execute
  ├── OWL → ambient intelligence (Bee)
  ├── ORB → voice TTS
  ├── INK → writing (Phase 2)
  ├── DEN → physical environment
  └── factory → patent drafting
  ↓
Execute MCP tool call
  ↓
Artifact Generation
  ├── Memory entries (arc.store → persistent memory)
  ├── Context state (esp.assemble → session context)
  └── Inline tool responses
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Route to the correct gadget based on trigger phrase | Any gadget trigger |
| `memory` | ARC gadget: store or search memories | `arc.store`, `arc.search`, memory trigger |
| `context` | ESP gadget: unified context assembly | `esp.assemble`, `/renew-ctx` |
| `mesh` | ANT gadget: fleet health via Tailscale | `ant.mesh`, mesh health |
| `browser` | TAP gadget: Chrome tabs + JS | `tap.tabs`, `tap.execute` |
| `ambient` | OWL gadget: Bee AI extraction | `owl.brief`, `owl.extract` |
| `voice` | ORB gadget: macOS TTS | `orb.say`, `orb.voices` |
| `environment` | DEN gadget: physical sensors | `den.environment` |
| `patent` | Factory gadget: patent pipeline | `factory.new`, `factory.draft` |
| `test` | Test a specific tool via stdio | `test gadget`, debugging |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Stored memories | ARC memory store (gadget-managed) | Persistent agent memory |
| Context state | ESP assembly output (session-scoped) | Unified context for current session |
| Patent drafts | `docs/patents/` via factory gadget | Filing-track drafts |

## Fallback Chain

1. **Primary:** shadow-mcp federation server via MCP protocol (stdio)
2. **MCP server not running:** Start via `bun run /Volumes/SHADOW/shadow-lab/apps/shadow-mcp/src/index.ts`
3. **Bun not available:** Report; suggest `brew install oven-sh/bun/bun`; fall back to direct CLI equivalents where possible
4. **ShadowArchive unmounted:** Run in `--local` mode (all mocks, no external deps); note degraded functionality
5. **Specific gadget fails:** Report gadget error; fall back to equivalent non-MCP tool (e.g., `arc.search` fails → `grep` across memory files)
6. **Last resort:** Describe what the gadget would do without executing; explicitly state no tool call was made

## Prerequisites

- `bun` runtime installed
- shadow-mcp source at `/Volumes/SHADOW/shadow-lab/apps/shadow-mcp/`
- MCP registration in `/Volumes/SHADOW/.mcp.json`
- For ARC: persistent memory store configured
- For ANT: Tailscale mesh active
- For OWL: Bee AI authenticated
- For TAP: Chrome with accessible tabs
- For DEN: macOS with sensor APIs

## Error Handling

| Failure | Recovery |
|---------|----------|
| MCP server won't start | Check bun version; check port conflicts; suggest `--local` mode |
| Tool call returns error | Parse error message; retry once; report if persistent |
| ARC memory store unavailable | Fall back to file-based memory; note degraded memory persistence |
| ANT can't reach Tailscale | Report Tailscale status; skip mesh health; note offline |
| OWL Bee not authenticated | Skip ambient extraction; report auth needed; suggest `bee` skill |
| DEN sensor read fails | Report which sensor failed; return available sensors only |
| TypeScript compile error in new gadget | Fix types before registering; do not deploy broken gadgets |

## Contract

- **Gadgets are tools, not agents.** Each gadget performs a specific function. Do not anthropomorphize or chain gadgets autonomously.
- **ARC memories are append-only.** Do not delete or overwrite existing memories via ARC.
- **TAP browser access respects operator profile boundaries.** Do not use TAP to access work URLs from personal Chrome profile or vice versa.
- **DEN sensor data is ephemeral.** Do not persist physical environment data without operator awareness.
- **No credential exposure.** MCP tool responses must not contain API keys or auth tokens in logs.
- **Externalization rule.** Patent drafts go to `docs/patents/`. Memory entries go to ARC store. No intermediate artifacts in `/tmp`.
- **Do not** register new gadgets without type-checking (`npx tsc --noEmit`).
- **Do not** use TAP to extract data from authenticated sessions the operator hasn't approved.
- **Do not** run factory gadget (patent generation) without operator approval — it consumes GLM/Ollama compute.
