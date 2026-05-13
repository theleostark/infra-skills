---
name: SHADOW Continuity
description: Use when exploratory research, architecture work, or multi-source synthesis should be continuously externalized into repo artifacts instead of staying in chat, with later promotion into runtime, skills, wrappers, or project packs.
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-continuity
  icon_style: craft-category-glyph-v1
---

# shadow-continuity

Continuous research-to-repo harness for `shadow-ent`, company wrappers, and project packs.

## Core Principle

Do not let important system design live only in conversation. Keep the loop running:

`discover -> structure -> stream to repo -> classify -> promote -> runtime consumes`

This is more than a meta-skill. It is a **meta-harness** for turning ongoing research into durable system memory and deployable structure.

## When to Use

- Large exploratory sessions
- Architecture design
- Reverse-engineering OSS / products / APIs
- Enterprise/company/project wrapper design
- Any session where multiple docs/YAML/scaffolds should be written as the work evolves

## What Gets Streamed

Write durable artifacts early into the repo:

- `docs/plans/` — architecture, reverse-engineering, merge/promotion notes
- YAML / JSON — canonical maps, version files, capability matrices, state machines
- scaffolds — runtime folders, package skeletons, config stubs
- skills — when the pattern is reusable beyond one repo doc

Do **not** dump raw chain-of-thought. Write:
- decisions
- rationale summary
- mappings
- contracts
- schemas
- plans

## Intake-to-Research Rule

If an inbound file contains a business ask, requirements packet, comparison target, or research trigger, do not stop at storage placement.

Do both in the same pass:

1. place the raw file into the canonical input lane
2. emit a short research brief or mapping note into the durable findings surface

Default heuristic:

- raw artifact -> program `inputs/` or `source-drops/`
- durable synthesis -> `ShadowArchive/80-reports/`

Bad pattern:

- moving the file to inbox
- stopping
- waiting for a second operator prompt before writing the research artifact

Good pattern:

- stage or promote the source
- immediately externalize the extracted ask, internal mapping, and next comparison step

## Harness Loop

### 1. Discover
- gather from chat, docs, runtime, source repos, observations, operator corrections

### 2. Structure
- classify into one of:
  - architecture
  - config/map
  - scaffold
  - skill
  - report

### 3. Stream to repo
- write the artifact immediately once the shape is stable enough

### 4. Classify
- decide target layer:
  - `shadow-ent`
  - company wrapper (e.g. `astemo-ent`)
  - project pack (e.g. `astemo-fh4s`)
  - compatibility alias / shim

### 5. Promote
- if stable and reusable, move from plan/note into:
  - skill
  - canonical YAML/JSON
  - runtime file

### 6. Runtime consume
- keep fixed system prompt small
- let runtime load repo truth from files instead of conversation memory

## Promotion Levels

### Level 1 — Note
Saved in `docs/plans/` or reports. Useful but not yet authoritative.

### Level 2 — Canonical map
Saved as YAML/JSON and treated as machine-readable truth.

### Level 3 — Skill
Saved as reusable operator/agent behavior.

### Level 4 — Runtime
Saved as scaffold/config/code actually consumed by the system.

## Layering Rule

Every streamed artifact should target the right layer:

- generic substrate -> `shadow-ent`
- company adaptation -> company wrapper
- project specifics -> project pack

Never let project-specific material silently harden inside the generic core.

## Output Rules

- user-facing docs: concise and structured
- machine-readable docs: YAML/JSON first when possible
- runtime scaffolds: minimal but real
- skills: reusable patterns only

## Quick Decision Table

| If the output is... | Write to... |
|---|---|
| exploratory architecture | `docs/plans/` |
| canonical mapping | YAML/JSON under base/ |
| reusable behavior | skill |
| immediately consumed by runtime | scaffold/config/code |

## Anti-Patterns

- leaving major architecture only in chat
- duplicating the same mapping in multiple docs
- promoting observed ideas directly to runtime without a canonical map
- stuffing everything into the system prompt instead of compiling from repo truth

## Strong Rule

`chat is discovery`

`repo is durable memory`

`runtime loads from repo`

## Pipeline

```
Input (exploratory research, architecture work, multi-source synthesis)
  ↓
Discover (gather from chat, docs, runtime, source repos, observations)
  ↓
Structure (classify: architecture, config/map, scaffold, skill, report)
  ↓
Stream to Repo (write artifact immediately once shape is stable)
  ↓
Classify Target Layer (shadow-ent, company wrapper, project pack, compat alias)
  ↓
Promote (note → canonical map → skill → runtime, as stability increases)
  ↓
Artifact Generation
  ├── docs/plans/           (Level 1 — exploratory notes)
  ├── YAML/JSON base/       (Level 2 — canonical maps)
  ├── skills/               (Level 3 — reusable behavior)
  ├── scaffold/config/code  (Level 4 — runtime consumed)
  └── ShadowArchive/80-reports/ (same-pass research artifacts)
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Continuous externalization: discover → structure → stream → classify → promote as work evolves | Any exploratory/architecture session |
| `checkpoint` | Single-pass externalization of current state | Mid-session checkpoint, before context compaction |
| `intake` | Inbound artifact → canonical placement + same-pass research brief | File/URL/email ingestion |
| `promote <artifact>` | Promote a specific artifact to the next level | `promote this to skill`, `promote to runtime` |
| `audit` | Check what should have been externalized but is still only in chat | `audit continuity`, `what's only in chat` |

## Artifact Routing

| Artifact Level | Path | Purpose |
|----------------|------|----------|
| Level 1 — Note | `docs/plans/` | Exploratory architecture, research notes |
| Level 2 — Canonical map | YAML/JSON under base directory | Machine-readable truth |
| Level 3 — Skill | `~/.agents/skills/<name>/SKILL.md` | Reusable agent behavior |
| Level 4 — Runtime | scaffold/config/code files | Actually consumed by the system |
| Research brief | `ShadowArchive/80-reports/` | Same-pass synthesis from inbound artifacts |

## Fallback Chain

1. **Primary:** Stream artifacts to repo paths as they stabilize → promote through levels
2. **Target path not writable:** Fall back to `ShadowArchive/80-reports/` for all outputs; note original target blocked
3. **Repo not on disk (ShadowArchive unmounted):** Buffer artifacts in session memory; write when volume mounts; note delayed externalization
4. **Company/project wrapper directory doesn't exist:** Create scaffold first, then stream; do not skip externalization due to missing directory
5. **Last resort:** Write minimal summary to chat with explicit note: "This should be externalized but no writable path was available. Re-run with volume mounted."

## Prerequisites

- Writable repo filesystem (local or ShadowArchive)
- `docs/plans/` directory or ability to create it
- Target layer directory structure (for company/project wrappers)
- Understanding of promotion levels (1–4)
- Access to `ShadowArchive/80-reports/` for research briefs

## Error Handling

| Failure | Recovery |
|---------|----------|
| Target path not found | Create directory; if parent not writable, fall back to `ShadowArchive/80-reports/` |
| Artifact too raw to classify | Write as Level 1 note; do not skip; classify later during promotion |
| Project-specific content leaked into generic core | Flag; propose move to project pack; do not auto-move |
| YAML/JSON syntax error in canonical map | Validate before writing; fall back to Markdown note if validation fails |
| Promotion target already exists with different content | Report conflict; do not overwrite; propose merge or versioning |

## Contract

- **Never let important system design live only in conversation.** This is the core rule.
- **Separate extraction from synthesis.** Source facts first, then system mapping, then final artifact.
- **Intake-to-research rule is non-negotiable.** Inbound artifacts with live asks produce both canonical placement AND a same-pass research brief. No two-prompt delays.
- **Layer correctly.** Generic substrate → shadow-ent. Company adaptation → company wrapper. Project specifics → project pack. Never let project material harden inside the generic core.
- **Externalization rule.** All durable artifacts go to repo paths. Chat is discovery; repo is memory; runtime loads from repo.
- **Do not** dump raw chain-of-thought into repo files. Write decisions, rationale summaries, mappings, contracts, schemas, and plans.
- **Do not** promote to runtime without a canonical map (Level 2) first.
- **Do not** skip the same-pass research brief for inbound enterprise artifacts.

## Crystallized From

- Session: 2026-04-16
- Original task: operator pushed toward streaming outputs directly into repo memory and then promoting the pattern into something beyond a simple meta-skill — a reusable research-to-repo harness spanning architecture, skills, runtime scaffolds, and company/project adaptation.
- Session: 2026-04-19
- Original task: an enterprise `.eml` from Downloads was first handled as storage intake, but the operator expected immediate expansion into research and canonical program placement.
- Pattern extracted: inbound enterprise artifacts that contain live asks or spec packets must produce both canonical placement and a same-pass research artifact.
