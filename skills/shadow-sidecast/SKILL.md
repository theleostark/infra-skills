---
name: SHADOW Sidecast
description: 'Use when the user needs: Global organizational broadcast system for sharing org relationships, cross-functional connections, and program insights across the Shadow ecosystem. Trigger on sidecast, broadcast org, share org chart, global org, team mapping, or cross-functional.'
icon: icon.svg
metadata:
  filePattern: []
  bashPattern:
  - sidecast
  - broadcast org
  - share org chart
  - global org
  - team mapping
  category: enterprise/fh4s
  family: shadow
  lifecycle: active
  canonical_slug: shadow-sidecast
  icon_style: craft-category-glyph-v1
---

# shadow-sidecast — Global Organizational Broadcast System

Broadcast organizational insights, relationships, and cross-functional connections across Shadow ecosystem for global-level project coordination.

## Purpose

Transform FH4S-centric workflows into global program management. Share org structure, team mappings, and cross-functional relationships with all agents, skills, and tools.

## Core Functions

### 1. Broadcast Org Structure
```bash
# Share organizational hierarchy with all Shadow systems
shadow-sidecast broadcast-org \
  --scope "usgrp1" \
  --include "reporting-lines,cross-functional,interaction-patterns"
```

**Output:** JSON manifest of:
- Reporting lines (manager → director relationships)
- Cross-functional counterparts (SQ, PE, BUP, PMO, Sourcing)
- Information flow patterns (US → Japan, cross-functional)
- Program assignments (who works on what)

### 2. Generate Team Mapping
```bash
# Create team mapping by program
shadow-sidecast team-map \
  --program "FH4S" \
  --output "org-map-fh4s.json"
```

**Output:**
- US team members + roles
- Japan counterparts + roles
- Cross-functional stakeholders + departments
- Meeting cadence by team

### 3. Share Global Priority Stack
```bash
# Broadcast priorities across all programs
shadow-sidecast priorities \
  --global \
  --programs "FH4S,FH4,FH4L,XP7G,XY9H" \
  --tags "P0,P1,P2"
```

**Output:** Priority stack with:
- Program tags (e.g., "[FH4S]", "[All Programs]")
- Owner assignments
- Cross-program dependencies
- Due dates

### 4. Detect Cross-Program Themes
```bash
# Analyze meeting transcripts for recurring themes
shadow-sidecast themes \
  --window "30d" \
  --min-frequency "3" \
  --output "theme-report.md"
```

**Output:** Weekly theme report showing:
- Recurring topics across programs
- Blockers affecting multiple programs
- Cross-program optimization opportunities

## Data Sources

### Primary (Canonical)
- **Org chart:** `/Volumes/☯Duality/ShadowArchive/20-ingested/dropbox/fh4s-ipm/org-chart.md`
- **CLAUDE.md:** `/Volumes/☯Duality/ShadowArchive/20-ingested/dropbox/fh4s-ipm/CLAUDE.md`
- **Meeting notes:** `/Volumes/☯Duality/ShadowArchive/20-ingested/dropbox/fh4s-ipm/07_Meeting_Notes/`
- **Smartsheet:** Workspace ID 4219108813236100

### Secondary (Derived)
- **Crystallization reports:** `/Volumes/☯Duality/ShadowArchive/80-reports/crystallize_org_relationship_map_2026-04-20.md`
- **Meeting transcripts:** Google Recorder, Teams, Bee AI
- **Action tracking:** Smartsheet Global Action Tracking (to be implemented)

## Output Formats

### JSON Schema (Machine-Readable)
```json
{
  "version": "1.0",
  "generated": "2026-04-20T12:00:00Z",
  "scope": "usgrp1",
  "org_structure": {
    "reporting_lines": [
      {
        "from": "Sabarish Duvvuru",
        "to": "Bryan Voris",
        "relationship": "reports_to"
      }
    ],
    "cross_functional": [
      {
        "department": "SQ",
        "contacts": ["Emily Howard", "Sakamoto Kenichi"],
        "programs": ["FH4S", "FH4", "FH4L", "XP7G", "XY9H"],
        "interaction_frequency": "bi-weekly"
      }
    ]
  }
}
```

### Markdown (Human-Readable)
```markdown
# Org Structure Broadcast — USGRP1

## Reporting Lines
- Sabarish → Bryan Voris → [Director Level]

## Cross-Functional Matrix
| Department | Contacts | Programs | Frequency |
...
```

## Integration Points

### 1. Skills Integration
**Target skills:** All `astemo-*` skills, `fh4s-*` skills, `m3-*` skills

**Broadcast to:** Each skill's context/kernel directory
```bash
# Update skill contexts with org structure
shadow-sidecast sync-skills \
  --skills "astemo-recorder,astemo-meeting-actions,fh4s-*" \
  --source "org-chart.md"
```

### 2. Smartsheet Integration
**Auto-populate:** "Global Action Tracking" sheet with:
- Owner names from org chart
- Program assignments
- Department tags
- Cross-program dependencies

### 3. Meeting Notes Enhancement
**Auto-tag:** Notes from all programs tagged with:
- Program codes (FH4S, FH4, FH4L, XP7G, XY9H)
- Department tags (DE, SQ, PE, BUP, PMO, Sourcing)
- Cross-program references (linked items)

### 4. Semantic Index Update
**Add to search index:** `/Volumes/☯Duality/ShadowArchive/40-datasets-models/semantic-index/`

**New documents to index:**
- Org chart (hierarchical structure)
- Cross-functional matrix (relational data)
- Global priority stack (action items)
- Meeting notes (all programs)

## Usage Patterns

### Pattern 1: Program-Agnostic Context
When any agent/skill needs to know "who works on what":
```bash
shadow-sidecast who --program "FH4L"  # Returns: Kevin Butler
shadow-sidecast who --role "SQ"      # Returns: Emily Howard, Sakamoto
shadow-sidecast who --manager        # Returns: Bryan Voris
```

### Pattern 2: Cross-Program Coordination
When action spans multiple programs:
```bash
shadow-sidecast cross-program --action "Resin coating decision"
# Returns: Affects FH4S, FH4 → Stakeholders: Hosoya, Lisa, Bryan
```

### Pattern 3: Meeting Preparation
Before any meeting, auto-generate attendee list:
```bash
shadow-sidecast attendees --meeting "FH4S DE Weekly"
# Returns: US (Sabarish, Kevin, Bryan), JP (Iwata, Hosoya, Sakamoto), Interpreter
```

## Implementation Priority

### Phase 1: Core Broadcast (Week 1)
1. Implement org structure parsing from org-chart.md
2. Create JSON schema for reporting lines + cross-functional matrix
3. Build CLI tools: `broadcast-org`, `team-map`, `priorities`

### Phase 2: Integration (Week 2)
1. Sync with Smartsheet Global Action Tracking
2. Update skill contexts with org data
3. Enhance meeting notes with program tagging

### Phase 3: Intelligence (Week 3-4)
1. Cross-program theme detection (NLP)
2. Resource capacity dashboard integration
3. Automated attendee list generation

## Success Metrics

- **Coverage:** All programs (FH4S, FH4, FH4L, XP7G, XY9H) indexed
- **Accuracy:** 100% of relationships from org chart captured
- **Latency:** Broadcast to all skills < 5 seconds
- **Adoption:** 80% of astemo-* skills use sidecast data within 2 weeks

## Related Skills

- `astemo-recorder` — Uses org data for speaker mapping
- `astemo-meeting-actions` — Uses global priority stack for action extraction
- `fh4s-*` skills — Consume program-specific org context
- `shadow-obsidian` — Broadcast to Obsidian vault widgets

## Maintenance

**Update triggers:**
- Org chart changes (quarterly or as-needed)
- New programs added (XY9H → add to all mappings)
- Team reorganizations (update reporting lines)
- Priority stack changes (daily auto-sync from Smartsheet)

**Manual refresh:**
```bash
shadow-sidecast refresh --source "org-chart.md" --force
```

## Pipeline

```
Input (trigger: sidecast, broadcast org, team mapping, cross-functional, who works on X)
  ↓
Load Data Sources:
  ├── Primary: org-chart.md, CLAUDE.md, meeting notes, Smartsheet
  └── Secondary: crystallization reports, meeting transcripts, action tracking
  ↓
Parse Org Structure (reporting lines, cross-functional, program assignments)
  ↓
Generate Output:
  ├── JSON manifest (machine-readable)
  ├── Markdown report (human-readable)
  └── Skill context updates (broadcast to astemo-*/fh4s-* skills)
  ↓
Artifact Generation
  ├── ShadowArchive/80-reports/sidecast-org-YYYY-MM-DD.json
  ├── ShadowArchive/80-reports/sidecast-org-YYYY-MM-DD.md
  └── Semantic index update
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Broadcast org structure across Shadow ecosystem | `sidecast`, `broadcast org` |
| `who` | Query: who works on a program/role/department | `who works on FH4S`, `who is SQ` |
| `team-map` | Team mapping for a specific program | `team map`, `team mapping` |
| `priorities` | Global priority stack across programs | `priorities`, `global priorities` |
| `themes` | Cross-program theme detection from meeting transcripts | `cross-program themes` |
| `attendees` | Auto-generate attendee list for a meeting | `meeting attendees`, `who should be in FH4S weekly` |
| `cross-program` | Find stakeholders affected by a cross-program action | `cross-program`, `who does this affect` |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Org JSON manifest | `ShadowArchive/80-reports/sidecast-org-YYYY-MM-DD.json` | Machine-readable org structure |
| Org Markdown report | `ShadowArchive/80-reports/sidecast-org-YYYY-MM-DD.md` | Human-readable org overview |
| Team map | `ShadowArchive/80-reports/team-map-<program>-YYYY-MM-DD.json` | Program-scoped team mapping |
| Theme report | `ShadowArchive/80-reports/theme-report-YYYY-MM-DD.md` | Cross-program theme analysis |
| Priority stack | `ShadowArchive/80-reports/priority-stack-YYYY-MM-DD.md` | Global priority overview |

## Fallback Chain

1. **Primary:** Parse org-chart.md + CLAUDE.md + Smartsheet → generate manifests → broadcast to skills
2. **Org chart unavailable:** Fall back to CLAUDE.md people tables + meeting notes extraction; note degraded org coverage
3. **Smartsheet unreachable:** Skip Smartsheet data; note that schedule/action data is stale
4. **Meeting notes empty:** Skip theme detection; report only structural org data
5. **Skill broadcast target not found:** Write JSON manifest; skills can self-load on next session; do not fail on broadcast delivery
6. **Last resort:** Return inline org summary from whatever sources are available; explicitly note coverage gaps

## Prerequisites

- Org chart accessible: `/Volumes/☯Duality/ShadowArchive/20-ingested/dropbox/fh4s-ipm/org-chart.md`
- CLAUDE.md accessible for people/program tables
- Smartsheet API token in keychain (optional, for schedule integration)
- `shadow-sidecast` CLI tool or equivalent inline logic
- Write access to `ShadowArchive/80-reports/` for manifests

## Error Handling

| Failure | Recovery |
|---------|----------|
| Org chart file not found | Check alternate paths; fall back to CLAUDE.md tables; note missing source |
| Smartsheet API 401 | Report auth failure; suggest keychain token refresh; continue without Smartsheet data |
| JSON parse error in org data | Fall back to raw text parsing; note data quality issue |
| Program code not recognized | List known programs (FH4S, FH4, FH4L, XP7G, XY9H); ask operator to clarify |
| Cross-functional contact missing | Report gap; do not invent contacts; flag for manual update |
| Semantic index write fails | Save manifest to `ShadowArchive/80-reports/` as fallback; index later |

## Contract

- **Read org data from canonical sources.** Do not invent or assume org relationships. If the org chart doesn't say it, don't report it.
- **Broadcast is informational.** Sidecast provides org context to skills and tools; it does not modify Smartsheet, SharePoint, or any enterprise system.
- **No credential exposure.** Org manifests must not contain SSO tokens, email passwords, or internal URLs that shouldn't be externalized.
- **Coverage honesty.** If a program or department is missing from the data, say so. Do not silently omit gaps.
- **Externalization rule.** All org manifests and reports go to `ShadowArchive/80-reports/`. Never leave the only org map in chat.
- **Do not** auto-update enterprise systems (Smartsheet, SharePoint) with sidecast data. Sidecast broadcasts to the agent ecosystem, not to Astemo infrastructure.
- **Do not** expose internal org structure in customer-facing or Dropbox-visible artifacts.
- **Do not** assume reporting lines that aren't explicitly documented.

---

**See also:**
- Crystallization report: `crystallize_org_relationship_map_2026-04-20.md`
- Implementation summary: `crystallize_org_map_implementation_summary_2026-04-20.md`
- CLAUDE.md (updated): Global priority stack + org relationships
