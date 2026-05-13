---
name: SHADOW Engg
description: 'Use when the user needs: Meta-exploration engine for the shadow-verse — discovers patterns, uncovers gaps, synthesizes tools, spawns sub-agents, maintains context across all shadow spaces. Also retains original Caresoft/iceberg engineering intelligence. Trigger on "shadow-engg", "explore", "discover", "what''s missing", "scan skills", "find patterns", "synthesize", "gap analysis", "verse report", "iceberg", "teardown", "engineering report".'
icon: icon.svg
metadata:
  depends_on: [skill-surgery-rd]
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-engg
  icon_style: craft-category-glyph-v1
---

# shadow-engg — Meta-Exploration Engine

The nervous system of the shadow-verse. Discovers what exists, what's missing, and what should be built.

## Skill surgery lane

When the exploration target is the skill garden itself, load `skill-surgery-rd` and use its audit → classify → surgery → research → validate loop. Default to repairing existing skills before creating new ones: redirect deprecated skills, repair weak trigger descriptions, trim oversized SKILL.md files into references, and create only when no canonical skill owns the lane.

## Two Modes

### Mode 1: Explore (meta, cross-domain)

Scan the entire shadow-verse — skills, memory, code, mesh — to find patterns, gaps, and opportunities.

```bash
# Scan all skills for patterns and gaps
shadow-engg scan

# Find what capabilities are missing
shadow-engg gaps

# Synthesize overlapping skills/tools
shadow-engg synthesize

# Deep-dive into a domain
shadow-engg explore security
shadow-engg explore infrastructure

# State-of-the-verse report
shadow-engg report
```

**Exploration targets:**
- `~/.Codex/skills/` — 100+ skills, pattern analysis
- `~/.Codex/projects/-Volumes-SHADOW/memory/` — 20+ memory files
- `/Volumes/SHADOW/ShadowArchive/` — managed archive structure
- `/Volumes/SHADOW/shadow-lab/apps/` — all shadow-* products
- Mesh nodes via SSH — FRIDAY, AURION, ULTRON state
- Feeds — X bookmarks, WhatsApp, Bee conversations

**What it discovers:**
- Duplicate/overlapping skills → merge candidates
- Domains with no skill coverage → creation targets
- Stale memory files → update/archive candidates
- Cross-domain patterns → new product/patent opportunities
- Mesh imbalances → workload redistribution suggestions

### Mode 2: Engineering Intelligence (original iceberg)

Automotive competitive intelligence via Caresoft API.

```bash
shadow-engg engineering-report <code> --profile inverter
shadow-engg head-to-head <codeA> <codeB>
shadow-engg fleet-dna --all
shadow-engg deep-mine --all
shadow-engg benchmarks
shadow-engg dispatch <code> --agent gemini
```

**Location:** `/Volumes/SHADOW/ASTEMO/tools/iceberg-cli/`
**Apex:** 18 modules, 52 tech signatures, 58-dim vehicle DNA
**Spark:** 6 AI backends, pipeline orchestrator
**Tests:** 406 passing

## Architecture (Apex/Spark pattern)

```
shadow-engg/
├── apex/              Exploration engine
│   ├── scanner        Scan skills, memory, code
│   ├── classifier     Classify by domain lane
│   ├── synthesizer    Merge overlapping tools
│   ├── gap-finder     Identify missing capabilities
│   └── reporter       Generate reports
├── spark/             Agent dispatch
│   ├── agents         Orchestrate across mesh
│   └── dispatch       Route to Codex/GLM/qwen3
├── connectors/
│   ├── mesh/          SSH to fleet nodes
│   ├── skills/        Introspect skill files
│   ├── memory/        Scan memory system
│   └── feeds/         X, WhatsApp, Bee
└── domains/
    ├── engineering/   Caresoft teardowns (original)
    ├── security/      Supply chain, CVE monitoring
    ├── product/       Pricing, market, competition
    └── infrastructure/ Mesh health, storage, services
```

## Relationship to Other Shadow Tools

| Tool | Role | shadow-engg's Relationship |
|------|------|---------------------------|
| shadow-cc | Identifies kernel vs derived | shadow-engg uses this to know WHAT exists |
| shadow-anchor | Saves session state | shadow-engg reads anchors to understand history |
| shadow-feed | Ingests external intelligence | shadow-engg classifies and routes feed items |
| shadow-top | Displays everything | shadow-engg's reports surface in shadow-top |
| shadow-kernel | The irreducible minimum | shadow-engg ensures the kernel is complete |

## The Metaphor

shadow-cc is the **immune system** — identifies foreign matter (derived) vs self (kernel).
shadow-engg is the **nervous system** — senses the environment, discovers patterns, coordinates response.
shadow-top is the **eyes** — displays what the nervous system discovers.
shadow-anchor is the **hippocampus** — stores memories for future retrieval.

## Pipeline

```
Input (trigger: shadow-engg, explore, discover, what's missing, gap analysis, engineering report)
  ↓
Classify Mode:
  ├── Explore (meta, cross-domain scan of skills/memory/code/mesh)
  └── Engineering Intelligence (automotive competitive via Caresoft)
  ↓
Scan Targets (skills, memory, code, mesh, feeds)
  ↓
Analyze:
  ├── Pattern detection (duplicates, gaps, stale content)
  ├── Cross-domain synthesis (product/patent opportunities)
  ├── Competitive intelligence (benchmarks, fleet DNA)
  └── Mesh health (imbalances, missing services)
  ↓
Artifact Generation
  ├── ShadowArchive/80-reports/engg-scan-YYYY-MM-DD.md
  ├── ShadowArchive/80-reports/engg-report-<topic>-YYYY-MM-DD.md
  └── Skill surgery proposals (route to skill-surgery-rd)
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Auto-detect: skill garden exploration | `shadow-engg`, `explore`, `discover` |
| `explore` | Full shadow-verse scan — skills, memory, code, mesh | `scan`, `verse report` |
| `gaps` | Gap analysis — what capabilities are missing | `what's missing`, `gap analysis` |
| `synthesize` | Merge overlapping tools/skills | `synthesize`, `find patterns` |
| `engineering` | Caresoft automotive competitive intelligence | `iceberg`, `teardown`, `engineering report` |
| `head-to-head` | Vehicle comparison | `head to head`, `compare vehicles` |
| `benchmarks` | Engineering benchmarks | `benchmarks`, `fleet-dna` |
| `report` | State-of-the-verse report | `verse report`, `state of the verse` |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Scan report | `ShadowArchive/80-reports/engg-scan-YYYY-MM-DD.md` | Cross-domain exploration results |
| Engineering report | `ShadowArchive/80-reports/engg-report-<topic>-YYYY-MM-DD.md` | Competitive intelligence |
| Gap analysis | `ShadowArchive/80-reports/engg-gaps-YYYY-MM-DD.md` | Missing capability inventory |
| Skill surgery proposals | Route to `skill-surgery-rd` skill | Skill garden improvements |

## Fallback Chain

1. **Primary:** Full scan across skills, memory, code, mesh → analyze → report
2. **ShadowArchive unmounted:** Scan local paths only (`~/.agents/skills/`, `~/.codex/`); note that archive scan was skipped
3. **Mesh unreachable:** Skip mesh health checks; note offline nodes; continue with local analysis
4. **Caresoft API unavailable:** Report; skip engineering intelligence mode; note that iceberg-cli is unreachable
5. **Skills directory empty:** Report; likely path issue; suggest checking `~/.agents/skills/`
6. **Last resort:** Minimal scan of current working directory only; explicitly state degraded coverage

## Prerequisites

- Access to skill directories (`~/.agents/skills/`, `.agents/skills/`)
- ShadowArchive volume mounted for full scans (optional for local-only mode)
- `iceberg-cli` at `/Volumes/SHADOW/ASTEMO/tools/iceberg-cli/` for engineering mode
- Tailscale mesh active for fleet scans (optional)
- `grep`, `find`, `wc` for pattern analysis
- Access to `ShadowArchive/80-reports/` for reports

## Error Handling

| Failure | Recovery |
|---------|----------|
| Skills directory not found | Check alternate paths; report; scan what's available |
| Memory directory empty | Report; note that no session history is available for analysis |
| Caresoft API returns error | Report API status; skip engineering mode; continue with exploration mode |
| Mesh SSH fails | Skip mesh health; report offline nodes; continue with local analysis |
| Report write fails | Output inline; note write failure; suggest mounting volume |
| Pattern extraction finds too many gaps (>50) | Prioritize top 10 by impact; list rest as count; avoid overwhelming output |

## Contract

- **Exploration is read-only.** shadow-engg scans and reports; it does not modify skills, code, or mesh configuration.
- **Skill surgery proposals are proposals.** Route to `skill-surgery-rd` for actual surgery; do not edit skills directly from shadow-engg.
- **Engineering intelligence is competitive analysis.** Do not share Caresoft teardown data outside SHADOW ecosystem.
- **No credential exposure.** Reports must not contain API keys, Caresoft tokens, or mesh SSH keys.
- **Externalization rule.** All scan and engineering reports go to `ShadowArchive/80-reports/`. Never leave the only analysis in chat.
- **Do not** auto-execute skill surgeries found during exploration. Report only.
- **Do not** expose Caresoft/iceberg competitive data in customer-facing or public artifacts.
- **Do not** modify mesh configuration based on scan results. Report imbalances; let operator decide.
