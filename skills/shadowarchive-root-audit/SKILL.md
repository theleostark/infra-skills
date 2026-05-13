---
name: 'ShadowArchive Root Audit'
description: Use when auditing a SHADOW or ShadowArchive-backed volume root and classify every top-level folder or file into canonical root docs, managed archive content, active workspaces, mirrors, installers, archives, or quarantine candidates. Use when a drive root is messy and needs a conservative storage truth model before any moves.
icon: icon.svg
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: shadowarchive-root-audit
  icon_style: craft-category-glyph-v1
---

Audit a SHADOW-style volume root before reorganizing it.

## Use when

- a drive root mixes research docs, workspaces, installers, archives, and managed archive content
- you need to know what should stay at root versus move under `ShadowArchive`
- multiple parallel-looking roots exist and you need a canonical map

## Workflow

1. Read the volume `AGENTS.md` first.
2. Inventory every root entry and major subtree.
3. Classify each item as:
   - root-canonical-doc
   - managed-shadowarchive
   - active-workspace
   - mirror-or-placeholder
   - archive-or-quarantine
   - installer-or-loose-binary
   - unknown-needs-triage
4. Record the proposed canonical home and action for each item.
5. Do not move anything until the audit is decision-complete.

## Safety

- No auto-delete.
- No auto-dedupe.
- No flattening mixed roots before classification.

## Pipeline

```
Input (volume root path or drive identifier)
  ↓
Read AGENTS.md (canonical layout expectations)
  ↓
Inventory (ls -la root, du -sh each subtree, file type detection)
  ↓
Classification (root-canonical-doc / managed-shadowarchive / active-workspace / mirror / archive / installer / unknown)
  ↓
Canonical Mapping (proposed home + action per item)
  ↓
Artifact Generation
  └── ShadowArchive/80-reports/root-audit-<volume-slug>-YYYY-MM-DD.md
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Full audit: inventory + classification + proposed actions for every root entry | Volume path given or "audit root" trigger |
| `summary` | One-line per root entry: name, classification, size | Quick overview |
| `triage` | Only unknown-needs-triage items with recommended actions | Focused cleanup session |
| `compare` | Diff two volume roots against each other | Multiple drives with overlapping content |
| `dry-run` | Show proposed moves/renames without executing | Confirmation step before reorganization |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Audit report | `ShadowArchive/80-reports/root-audit-<volume-slug>-YYYY-MM-DD.md` | Full classification + proposed actions |
| Triage report | `ShadowArchive/80-reports/root-triage-<volume-slug>-YYYY-MM-DD.md` | Only items needing action |
| Compare report | `ShadowArchive/80-reports/root-compare-YYYY-MM-DD.md` | Multi-volume overlap analysis |

## Fallback Chain

1. **Primary:** Read `AGENTS.md` → `ls -la` + `du -sh` + file type detection → classify → propose
2. **No AGENTS.md:** Proceed with classification based on SHADOW conventions (see ARCHITECTURE.md); note that AGENTS.md was missing
3. **Volume unmounted:** Report volume offline; cannot audit; ask operator to mount
4. **Permission denied on some entries:** Classifiy accessible entries; mark denied entries as `unknown-permission-denied`; do not `sudo`
5. **Last resort:** Bare `ls` with no classification; explicitly state that the audit is incomplete

## Prerequisites

- Target volume mounted and readable
- `du`, `ls`, `file` commands available
- Access to `ShadowArchive/80-reports/` for writing the audit report
- AGENTS.md at volume root (optional, but improves classification accuracy)
- Reference to `ARCHITECTURE.md` or `SHADOW-ARCHITECTURE-SPEC.md` for canonical layout expectations

## Error Handling

| Failure | Recovery |
|---------|----------|
| Volume not found | Report; list mounted volumes; suggest correct path |
| AGENTS.md missing | Proceed without it; note in report; classification may be less accurate |
| Symlink loop detected | Mark as `unknown-symlink-loop`; skip subtree; do not follow |
| Extremely large tree (`du` times out) | Sample top-level only; estimate subtree sizes; note incomplete sizing |
| Mixed case / unicode path issues | Normalize paths in report; flag for manual review |
| Read-only volume | Classify what's readable; note read-only status; propose no changes |

## Contract

- **No auto-delete.** Audit classifies and proposes; it never moves, renames, or deletes files.
- **No auto-dedupe.** Duplicate detection is informational only.
- **No auto-flatten.** Mixed roots are flagged, not reorganized.
- **Read-only operation.** Root audit must not modify the volume in any way. The only writes are to the audit report in `ShadowArchive/80-reports/`.
- **Decision-complete before action.** The audit report must be reviewed and approved before any reorganization skill (tiering, offload) acts on it.
- **Externalization rule.** All audit reports go to `ShadowArchive/80-reports/`. Never leave the only classification in chat.
- **Do not** execute proposed actions from within the audit skill. Emit the report and stop.
- **Do not** classify content by reading file bodies. Classify by path, name, extension, and metadata only.
