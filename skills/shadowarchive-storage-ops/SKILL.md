---
name: 'ShadowArchive Storage Ops'
description: 'Use when the user needs: Coordinate ShadowArchive root audit, storage tiering, and offload workflows for SHADOW-backed drives and low-space laptops. Use when the user needs a complete storage cleanup and canonicalization flow rather than a one-off move.'
icon: icon.svg
metadata:
  depends_on: [shadowarchive-offload,shadowarchive-root-audit,shadowarchive-tiering]
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: shadowarchive-storage-ops
  icon_style: craft-category-glyph-v1
  replaced_by: canonical
---

Coordinate SHADOW storage cleanup as a staged workflow.

## Use these skills in order

1. `shadowarchive-root-audit`
2. `shadowarchive-tiering`
3. `shadowarchive-offload`

## Workflow

1. Audit the volume root and active workspace roots.
2. Assign hot, warm, and cold placement.
3. Choose the lowest-risk first-wave offload targets.
4. Move or mirror only after classification is complete.
5. Keep the root curated with only canonical docs and a few entrypoint symlinks.

## Rules

- Keep moves reversible where possible.
- Prefer canonical destinations under `ShadowArchive`.
- Use reports and manifests before broad cleanup.

## Pipeline

```
Input (volume path, drive identifier, or "clean up storage" trigger)
  ↓
Phase 1: Root Audit (shadowarchive-root-audit)
  ├── Read AGENTS.md
  ├── Inventory root entries
  └── Classify each item
  ↓
Phase 2: Tiering (shadowarchive-tiering)
  ├── Measure local pressure
  ├── Assign hot / warm / cold placement
  └── Prioritize offload candidates
  ↓
Phase 3: Offload (shadowarchive-offload)
  ├── Copy → validate → symlink (lowest-risk first)
  └── Generate manifest per move
  ↓
Phase 4: Verification
  ├── Root is curated (canonical docs + entrypoints only)
  ├── Disk space reclaimed verified
  └── All symlinks resolve
  ↓
Artifact Generation
  ├── ShadowArchive/80-reports/storage-ops-<volume-slug>-YYYY-MM-DD.md  (master report)
  ├── ShadowArchive/80-reports/storage-ops-manifest-YYYY-MM-DD.md       (all moves)
  └── Per-phase reports from sub-skills
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Full staged workflow: audit → tier → offload → verify | "clean up storage", "storage ops" trigger |
| `audit-only` | Run Phase 1 only; classify and report without acting | Assessment before committing |
| `plan` | Show proposed tiering + offload targets with sizes; no moves | Confirmation step |
| `execute` | Run the full pipeline after plan approval | Explicit go-ahead |
| `status` | Report current disk usage, tiering state, and last cleanup date | Quick health check |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Master storage report | `ShadowArchive/80-reports/storage-ops-<volume>-YYYY-MM-DD.md` | Full pipeline summary |
| Move manifest | `ShadowArchive/80-reports/storage-ops-manifest-YYYY-MM-DD.md` | Every move with source → destination |
| Phase sub-reports | `ShadowArchive/80-reports/` (per sub-skill naming) | Audit, tiering, offload details |

## Fallback Chain

1. **Primary:** Full staged pipeline — audit → tier → offload → verify
2. **Sub-skill unavailable:** Run the equivalent logic inline (e.g., if `shadowarchive-root-audit` skill is not loadable, perform root inventory directly)
3. **Volume unmounted:** Report which volumes are online; audit available volumes only; note offline volumes
4. **Offload destination full:** Stop at tiering phase; report what needs to move but cannot; propose alternative destinations
5. **Partial failure (some moves fail):** Complete successful moves; report failures individually; do not rollback successful moves
6. **Last resort:** Audit-only mode; generate report with proposed actions for manual execution later

## Prerequisites

- At least one SHADOW/ShadowArchive volume mounted and writable for offload targets
- Source volume readable
- Sub-skills available: `shadowarchive-root-audit`, `shadowarchive-tiering`, `shadowarchive-offload` (or equivalent inline logic)
- `du`, `df`, `rsync`, `ln` available
- Write access to `ShadowArchive/80-reports/`
- AGENTS.md on target volume (improves audit accuracy)

## Error Handling

| Failure | Recovery |
|---------|----------|
| Volume unmounted at start | List mounted volumes; ask operator to mount target; proceed with available volumes |
| Volume unmounts mid-pipeline | Stop pipeline; report last completed phase; do not attempt to continue |
| Sub-skill crashes | Run equivalent inline; note degraded mode in report |
| rsync fails | Fall back to `cp -a`; note in manifest |
| Disk full during offload | Abort that offload; continue with smaller targets; report all failures |
| Symlink creation fails | Keep both copies; report for manual resolution; do not delete original |
| Validation mismatch | Do not consider that offload complete; flag for manual review |

## Contract

- **Staged execution.** Never skip phases. Audit must complete before tiering; tiering before offload.
- **No auto-delete.** The pipeline never deletes originals without explicit operator confirmation after validation.
- **Reversibility.** Every offload must have a rollback path. Storage ops without rollback records is a bug.
- **Report before action.** Always generate the plan/report before executing moves. The operator must approve.
- **Volume integrity.** Never write to a volume with <5% free space. Check before each write phase.
- **Sub-skill isolation.** Each phase is independently recoverable. A failure in offload does not invalidate the audit or tiering results.
- **Externalization rule.** All reports and manifests go to `ShadowArchive/80-reports/`. Never leave the only pipeline record in chat or `/tmp`.
- **Do not** clean caches (node_modules, .cache) as part of storage-ops. That is a separate `cache clean` operation with its own safety rules.
- **Do not** modify Dropbox, iCloud, or OneDrive sync roots. Those are managed by their own agents.
