# SP Allocation Map — Complete (Deep Mined 2026-05-03)

**Source:** Full ecosystem grep across Duality, skills, registry, reports, sessions  
**Method:** `grep -rn "SP-0"` across all .md/.json/.tex surfaces  
**Purpose:** Single source of truth — no more collisions

---

## Filing Track (Patent Registry)

| SP | Title | Status | Artifacts |
|----|-------|--------|-----------|
| SP-001 | Physical-Presence-Driven Context Management with Temperature-Tiered Memory | DRAFT COMPLETE | `docs/patents/SP-001/` |
| SP-002 | LASER Amplification for AI Agents | THESIS CAPTURED | `shadowlab_laser_thesis.md` |
| SP-003 | Agent Access Control with Human-in-the-Loop Approval | DRAFTING | — |
| SP-011 | Three-Layer Context Routing for Personal AI Mesh Networks | DRAFTING | `docs/patents/SP-011/` |
| SP-012 | Device-Aware Model Quantization Distribution for Heterogeneous AI Meshes | DRAFTING | `docs/patents/SP-012/` |
| SP-013 | Persona-Aware Generative Capability Synthesis (GLI) | DRAFT COMPLETE | `docs/patents/SP-013/` |
| SP-014 | Identity-Bound Model Routing / Persona-Bound Inference (PBI) | DRAFT COMPLETE | `docs/patents/SP-014/` |
| SP-015 | Dual-Mode Addressing for AI Agent Context Management Across Heterogeneous Display Surfaces | DRAFT NEAR FILING | `docs/patents/SP-015/` |
| SP-018 | Continuous Configuration Evolution for Personalized AI Model Training | RESEARCH DRAFT | `ShadowArchive/20-archives/shadow-research/patents/` |
| SP-019 | Virtual VRAM with Mesh-Tiered Offload for Distributed Neural Networks | RESEARCH DRAFT | `ShadowArchive/20-archives/shadow-research/patents/` |
| SP-024 | Privacy-Bounded Agentic Synthesis from Local Operating-System Metadata | DRAFTING / FILING | `docs/patents/SP-024/` |
| SP-025 | Privacy-Enumerated Context Compression for Agentic Runtime Enrichment | REGISTERED (SP-024 continuation) | — |
| SP-026 | Agentic Public-Domain Patent Mining and Commercial Viability Scoring | CANDIDATE ONLY — block filing | — |

## Referenced-Only Track (Protocol / Skill / Principle)

| SP | Title | Namespace | Active In |
|----|-------|-----------|-----------|
| SP-004 | Temporal Anchoring | protocol | shadow-anchor, shadow-cast |
| SP-005 | Kernel / Dehydration / Rehydration | protocol | shadow-intel, SP-005 Obsidian note |
| SP-006 | Latent Skill Observer (PostToolUse) | protocol | sp006-observer, crystallize, gli, 13 latent skills |
| SP-007 | Mesh / Distributed (CLI→MCP Bridge) | protocol | shadow-pm, shadow-harness |
| SP-008 | Inference Optimization | protocol | shadow-intel |
| SP-009 | Kernel Morphism | referenced-only | synthesis report |
| SP-010 | Shadow Pair / Cryptographic Kernel-SLM Binding | protocol | shadow-intel |
| SP-016 | Firmware-Aligned Context Operating System | protocol | shadow-intel, shadow-lists |
| SP-017 | Kernel Whisper (on-device ternary GPT for e-paper) | protocol | ECHO product roadmap, SP-018/019 specs |
| SP-020 | Voice/E-Paper Retrieval (split from WebSocket) | referenced-only | collision resolution report |
| SP-021 | Adaptive Display Mode Switching (accelerometer + ambient light) | referenced-only | synthesis report |
| SP-022 | Ambient Companion Device (split from SP-019) | referenced-only | collision resolution, strongest promotion candidate |
| SP-023 | WebSocket Mode (split from SP-020) | referenced-only | collision resolution, strongest promotion candidate |
| **SP-027** | **Visual Verifiability for Rendered Artifacts** | **principle** | **shadow-typora-engine, VERIFIABILITY.md, astemo.css** |

## Gaps (no references found)

SP-028+

## Collision History

| Date | Wrong | Right | Lesson |
|------|-------|-------|--------|
| 2026-04-21 | SP-019 overloaded | Split to SP-019 + SP-022 | Title collision resolution |
| 2026-04-21 | SP-020 overloaded | Split to SP-020 + SP-023 | Title collision resolution |
| 2026-05-03 | SP-016 (mine) | SP-027 | Didn't check existing allocation |
| 2026-05-03 | SP-026 (mine) | SP-027 | Didn't check referenced-only track |

## Allocation Rule

**Before assigning any SP number:** grep the FULL ecosystem (skills, registry, archive, reports, sessions, Obsidian). Check both filing track AND referenced-only track. The patent registry only shows filing-track items — referenced-only items are invisible there.

```bash
find /Volumes/☯Duality /Users/sdluffy/.agents/skills -name "*.md" -o -name "*.json" | xargs grep -oh "SP-[0-9][0-9]*" | sort -u
```

---

*Deep mined from all surfaces. Every SP-001 through SP-027 accounted for. No gaps above 027.*
