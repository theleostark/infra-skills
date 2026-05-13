---
name: SHADOW Pi
description: 'Use when the user needs: Pi monorepo integration — coding agent, unified AI API, TUI library, GPU pods, Slack bot, DOE harness. Located on AURION at /data/ShadowArchive/10-projects/pi-mono-doe/. Trigger on "pi", "coding agent", "pi-mono", "pi-ai", "pods", "GPU", "vLLM", "DOE", "harness", "pi TUI".'
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-pi
  icon_style: craft-category-glyph-v1
---

# shadow-pi — Pi Monorepo Integration

The pi-mono-doe project on AURION contains 8 packages that form a complete AI agent stack.

## Location

AURION: `/data/ShadowArchive/10-projects/pi-mono-doe/`
Access: `ssh aurion-ts` then `cd /data/ShadowArchive/10-projects/pi-mono-doe/`

## Packages

| Package | Purpose | Shadow-Verse Integration |
|---------|---------|------------------------|
| `agent` | General-purpose agent framework | Base for all shadow-* agents |
| `ai` | Unified LLM API (auto model discovery) | Replace per-provider SDK calls |
| `coding-agent` | CLI with read/bash/edit/write | shadow-code foundation |
| `doe` | Design of Experiments harness | Testing + optimization |
| `mom` | Slack bot → agent delegation | shadow-bot integration |
| `pods` | vLLM GPU pod management | dexo distributed inference |
| `tui` | Terminal UI (differential rendering) | shadow-top enhancement |
| `web-ui` | AI chat components | shadowlab.cc frontend |

## Key Commands

```bash
# On AURION
ssh aurion-ts
cd /data/ShadowArchive/10-projects/pi-mono-doe

# Run coding agent
npx pi-coding-agent

# List packages
ls packages/

# Run tests
npm test
```

## Harness Engineering Pattern

The DOE harness + shadow-engg's apex/harness.js share the same 5-mechanism pattern:
1. Contracts (input/output validation)
2. Circuit Breaker (cascade prevention)
3. Health Check (integrity verification)
4. Invariant Guard (data consistency)
5. Telemetry (timing + error rates)

Apply this pattern to ALL shadow-* products for production reliability.

## CRUD Operations

The coding-agent package provides standard CRUD via tools:
- `read` — file reading with line ranges
- `write` — file creation
- `edit` — surgical edits (like Codex's Edit tool)
- `bash` — command execution

This is the foundation for a `shadow-crud` skill that standardizes data operations across the verse.


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

