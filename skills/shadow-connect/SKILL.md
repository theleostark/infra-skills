---
name: SHADOW Connect
description: 'Use when the user needs: Connecting layer between Codex 4.6 Opus and the Shadow Lab mesh. Three-layer architecture: User Layer (profiles, zones, preferences), Data Layer (kernel, memory), Compute Layer (device registry, model routing). Context zone enforcement (Enterprise/Personal/Developer), auto-dispatch, session continuity. Activates on session start. Use when routing queries to local vs cloud, switching context zones, coordinating mesh resources, or when auto-mode needs dispatch guidance.'
icon: icon.svg
allowed-tools: Bash(*), Read, Write
metadata:
  shadow_family: infrastructure
  shadow_role: orchestrator
  shadow_scope: system
  capability_summary: Mesh routing, context zones, session continuity, auto-dispatch
  openclaw_surfaces:
  - codex
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-connect
  icon_style: craft-category-glyph-v1
---

# shadow-connect

The connecting layer between Codex (cloud LLM) and the Shadow Lab mesh (local infrastructure). Implements three-layer separation with context zone isolation.

## Three-Layer Architecture

```
USER LAYER (this skill)          → trains SLM behavior (routing.jsonl)
  Profiles, zones, preferences
  Config: ~/.shadow-connect/profiles.yaml

DATA LAYER (kernel/memory)       → feeds SLM knowledge (instructions.jsonl)
  Skills, memory, facts, JSONL
  Location: ~/.Codex/skills/, ~/.Codex/projects/-Volumes-SHADOW/memory/

COMPUTE LAYER (mesh)             → executes SLMs per device
  Model registry, device specs
  Config: /Volumes/SHADOW/ShadowArchive/10-projects/ANE-LM/model-registry.yaml
```

## Context Zones

### Zone Detection

On every prompt, detect the active zone by keyword matching:

| Zone | Trigger Keywords | Priority |
|------|-----------------|----------|
| **Enterprise** | FH4S, IPM, SASG, Honda, Astemo, inverter, xEV, DFMEA, DRBFM, change point, Greenfield, Tarboro, 4E0A | Highest |
| **Developer** | shadow-*, mesh, fleet, deploy, build, skill, kernel, ANE, compile, spec, patent, SP-0 | Medium |
| **Personal** | Everything else (default) | Lowest |

Enterprise zone takes priority. If both enterprise and developer keywords appear, use Enterprise.

### Zone Isolation Rules

**Enterprise (locked)**
- Memory access: ONLY `astemo_*`, `fh4s_*`, `sasg_*`, `feedback_company_name`
- Model routing: Cloud Codex ONLY — NDA content must NEVER touch local models
- **GLM-001: GLM/Z.AI BLOCKED in enterprise zone** — no fallback to glm-global-coding-plan under any circumstance. If Codex is at cap, route to local Ollama or wait. Chinese-hosted API has no Astemo NDA coverage.
- GLM tier gate: Even if data appears GREEN, enterprise zone context carries RED residue. Treat all enterprise zone traffic as RED for GLM routing purposes.
- Skills: fh4s-*, astemo-*, upscaler-*, fh4s-meeting-prep, fh4s-honda-response
- DO NOT mention Shadow Lab products, personal projects, or mesh infrastructure
- Company name is "Astemo" (never "Hitachi Astemo")
- Perspective: Sabarish Duvvuru, US DE Lead at USGRP1

**Developer (open)**
- Memory access: `shadow*`, `fleet*`, `mesh*`, `patent*`, `context*`, `experience*`
- Model routing: Local SLM for classification → cloud for complex synthesis
- Skills: shadow-*, mesh-*, hl-*, patent-queue, shadow-engg, nnn-cli
- Full mesh visibility, can dispatch to any node

**Personal (open, default)**
- Memory access: All except enterprise-only files
- Model routing: GLM preferred (GREEN tier) → local SLM → cloud fallback
- **GLM-001: GLM allowed for GREEN/AMBER tier data.** Classify before routing. RED-tier content (even in personal zone) still blocks GLM.
- Skills: All non-enterprise skills
- Full access to shadow-feed, social skills, personal tools

### Zone Switching

Zones can change mid-conversation. When enterprise keywords appear:
1. Announce zone switch: "Switching to Enterprise zone"
2. Restrict memory reads immediately
3. Do not reference any personal/developer context from earlier in conversation
4. When enterprise keywords stop appearing for 3+ prompts, offer to switch back

## Auto-Dispatch Rules

When the user's prompt matches a pattern, auto-fire the appropriate skill:

| Pattern | Dispatch To | Zone |
|---------|------------|------|
| URL in prompt | hl-fetch (or specific hl-* if recognized source) | Any |
| `news.ycombinator.com` | hl-hn | Personal |
| `x.com`, `twitter.com` | hl-x | Personal |
| "analyze this X account", "who is this on X", "track claims" | x-research | Personal |
| "grow on X", "thread strategy", "posting strategy", "content pillars" | x-growth | Personal |
| "write this for X", "turn this into a thread", "reply pack" | x-content-lab | Personal |
| "how does X work", "should this be a thread", "creator workflow" | x-operator | Personal |
| "verify this X claim", "is this benchmark real", "claim ledger" | x-claims | Personal |
| "watch this topic on X", "monitor this account", "daily X digest" | x-watch or shadow-feed | Personal |
| `youtube.com` | hl-youtube | Personal |
| `github.com` (trending/notifications) | hl-github | Developer |
| "what's new", "check feed" | shadow-feed | Personal |
| "mesh status", "fleet health" | mesh-status | Developer |
| "patent", "SP-0*", "IP portfolio" | patent-queue | Developer |
| "FH4S", "change point", "Honda" | fh4s-context + zone switch | Enterprise |
| "meeting prep", "agenda" | fh4s-meeting-prep | Enterprise |
| "build", "create", "implement" | brainstorming skill first | Developer |
| "dashboard", "visualize" | shadow-dashboard | Developer |
| "check phone", "ADB" | shadow-adb | Personal |

## Routing Intelligence

### Model Selection (when SLMs are available)

```
Query arrives →
  1. Zone detection (keyword match)
  2. Data tier classification (GLM-001):
     - RED markers detected → tier=RED
     - AMBER markers detected → tier=AMBER
     - No markers → tier=GREEN
  3. GLM gate check:
     - Enterprise zone → GLM BLOCKED (even for GREEN tier)
     - RED tier → GLM BLOCKED
     - AMBER tier → GLM allowed, prefer Gemini first
     - GREEN tier → GLM preferred (maximize subscription)
  4. Complexity assessment:
     - Simple lookup → local slm-micro on JARVIS (ANE)
     - Classification → local slm-pico on JARVIS (ANE)
     - Decomposition → local slm-nano → fan out sub-queries
     - Complex/novel → cloud Codex 4.6 Opus
     - Enterprise zone → ALWAYS cloud Codex (NDA isolation)
  5. Device selection (from model-registry.yaml):
     - JARVIS available + ANE idle → JARVIS
     - JARVIS busy → FRIDAY (Ollama)
     - Batch workload → AURION
  6. Fallback: local SLM → FRIDAY Ollama → cloud Codex
     - NOTE: GLM is NOT in the fallback chain for enterprise/RED traffic
     - GLM IS the preferred primary for GREEN personal/dev traffic
```

### Pre-SLM Mode (current state)

Until slm-pico/nano/micro are trained and deployed:
- All queries route to cloud Codex (current behavior)
- Zone detection and skill dispatch still work
- Model registry is read-only reference for future routing

## User Profile Layer

Config file: `~/.shadow-connect/profiles.yaml`

```yaml
zones:
  enterprise:
    locked: true
    model_routing: cloud_only
    memory_scope: [astemo_*, fh4s_*, sasg_*]
  personal:
    locked: false
    model_routing: local_preferred
    memory_scope: all
  developer:
    locked: false
    model_routing: local_classify_cloud_complex
    memory_scope: [shadow*, fleet*, mesh*, patent*]

preferences:
  response_style: concise
  auto_dispatch: true
  default_zone: personal
  auto_mode: true

schedule:
  morning: shadow-feed + upscaler-brief
  pre_meeting: fh4s-meeting-prep (if FH4S meeting on calendar)
  end_of_day: shadow-anchor
```

User profile data feeds `routing.jsonl` for SLM training — the model learns your routing preferences over time.

## Session Continuity

### On SessionStart
1. Read last shadow-anchor (if exists)
2. Check fleet status: `tailscale status --json | python3 -c "..."`
3. Read model-registry.yaml for available SLMs
4. Detect zone from initial prompt or last anchor
5. Load appropriate memory subset

### On PreCompact
1. Capture kernel hash (shadow-research hook)
2. Extract training pairs from conversation
3. Write to `~/.shadow-research/queue/kernel-diff.jsonl`

### On Session End
1. Write temporal anchor via shadow-anchor
2. Record zone + routing decisions for training data
3. Update `estimates.jsonl` with actual session metrics

## Integration Points

- **shadow-research**: Kernel diffs feed Phase 1 crawler, training pairs feed Phase 3 trainer
- **shadow-feed**: Auto-dispatched for social content ingestion
- **hl-fetch**: Auto-dispatched for arbitrary URL extraction
- **model-registry.yaml**: Read for routing decisions, updated by shadow-research Phase 3
- **shadow-anchor**: Session persistence across conversations
- **fleet_registry.md**: Device capabilities for routing

## Training Data Generation

Every routing decision made by shadow-connect generates training data:

```jsonl
{"query_pattern": "check FH4S status", "detected_zone": "enterprise", "dispatched_skill": "fh4s-context", "model_route": "cloud", "correct": true}
{"query_pattern": "fetch arxiv paper", "detected_zone": "personal", "dispatched_skill": "hl-fetch", "model_route": "local_slm", "correct": true}
```

This feeds `routing.jsonl` → slm-nano training → better future routing.
