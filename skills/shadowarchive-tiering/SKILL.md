---
name: 'ShadowArchive Tiering'
description: Use when applying a hot, warm, and cold storage model across local SSD and SHADOW-backed volumes. Use when internal disk is under pressure and you need a lean-first placement policy for repos, corpuses, archives, experiments, and work outputs.
icon: icon.svg
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: experimental
  canonical_slug: shadowarchive-tiering
  icon_style: craft-category-glyph-v1
  replaced_by: offloading
---

Apply a lean-first storage tiering policy.

## Use when

- laptop storage is low
- a SHADOW or ShadowArchive volume has spare capacity
- you need to decide what stays local versus what moves to external storage

## Default model

- `hot`: local SSD only
  - active repos
  - runtime state
  - current sprint work
- `warm`: canonical on SHADOW, reachable via symlink or wrapper
  - active corpuses
  - review outputs
  - large references
- `cold`: SHADOW only
  - experimental trees
  - dormant projects
  - installers
  - old exports

## Workflow

1. Measure local pressure and the largest local consumers.
2. Assign each major tree to hot, warm, or cold.
3. Prefer offloading large low-risk trees first.
4. Keep runtime dotdirs local.
5. Use canonical destinations plus symlink entrypoints instead of duplicated copies.

## Safety

- Do not move runtime state off the local SSD.
- Do not move active repos blindly.
- Treat `experimental` as cold by default unless the user says it is active.

## Pipeline

```
Input (volume path or "tier storage" trigger)
  ↓
Measure Local Pressure (df -h boot volume, identify largest consumers via du)
  ↓
Inventory Major Trees (du -sh top-level dirs, classify by activity)
  ↓
Tier Assignment:
  ├── hot (local SSD) — active repos, runtime state, current sprint
  ├── warm (SHADOW + symlink) — active corpuses, review outputs, large refs
  └── cold (SHADOW only) — experimental, dormant projects, installers, old exports
  ↓
Prioritization (largest + lowest-risk first for offload candidates)
  ↓
Artifact Generation
  ├── ShadowArchive/80-reports/tiering-plan-<volume>-YYYY-MM-DD.md
  └── ShadowArchive/80-reports/tiering-executed-YYYY-MM-DD.md  (after moves)
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Tiering plan: classify all major trees + propose hot/warm/cold assignments | `tier`, `tiering` trigger |
| `assign <path>` | Classify a single path and recommend its tier | Specific path given |
| `estimate` | Report how much space each tier change would reclaim | `estimate tiering`, sizing question |
| `execute` | Apply the tiering plan (offload warm/cold items) | After plan approval |
| `status` | Show current tiering state: what's hot/warm/cold | Quick check |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Tiering plan | `ShadowArchive/80-reports/tiering-plan-<volume>-YYYY-MM-DD.md` | Proposed assignments before execution |
| Execution report | `ShadowArchive/80-reports/tiering-executed-YYYY-MM-DD.md` | What was actually moved and where |
| Size estimates | `ShadowArchive/80-reports/tiering-estimate-YYYY-MM-DD.md` | Reclaimable space by tier |

## Fallback Chain

1. **Primary:** `du` + `df` analysis → classify → propose tiering plan
2. **SHADOW volume unmounted:** Generate tiering plan with local data only; mark warm/cold destinations as pending volume mount; do not execute
3. **`du` times out on large tree:** Sample top-level sizes; note incomplete; proceed with available data
4. **Insufficient SHADOW capacity:** Report capacity gap; propose partial offload or alternative destinations
5. **Last resort:** Tiering analysis without execution — report only, explicitly note that no moves were made

## Prerequisites

- macOS or Linux with `df`, `du`, `ls`
- SHADOW/ShadowArchive volume mounted (required for warm/cold destinations; optional for plan-only mode)
- Write access to `ShadowArchive/80-reports/` for tiering artifacts
- `rsync` or `cp -a` + `ln -s` for execution phase
- `offload_rules.md` if site-specific rules exist

## Error Handling

| Failure | Recovery |
|---------|----------|
| Volume unmounted | Plan with available data; note pending destinations; do not execute |
| `du` timeout | Sample; estimate; note incomplete sizing in report |
| Runtime state misclassified as cold | Safety check rejects runtime dotdirs; flag for manual review |
| Active repo detected in cold candidate | Promote to warm or hot; do not offload without confirmation |
| Permission denied on tree | Skip; report as `unknown-permission-denied`; do not `sudo` |
| SHADOW destination full | Stop execution; report capacity; propose alternatives |

## Contract

- **No auto-move.** Tiering proposes; it does not execute moves unless the operator explicitly approves the plan.
- **Runtime state stays local.** Runtime dotdirs (`.ssh`, `.config`, `.local`, `.npm`, `.cache` for active tools) are always hot. This is non-negotiable.
- **Active repos stay hot unless explicitly approved.** The skill may flag a repo as warm/cold candidate, but must not execute without confirmation.
- **`experimental` is cold by default.** Unless the operator says otherwise.
- **Reversibility.** Any warm-tier offload must produce a rollback record.
- **Volume integrity.** Never fill a SHADOW volume past 95% capacity.
- **Externalization rule.** All tiering plans and execution reports go to `ShadowArchive/80-reports/`. Never leave the only tiering record in chat.
- **Do not** combine tiering with cache cleanup. Caches are a separate operation.
- **Do not** tier Dropbox, iCloud, OneDrive, or Google Drive sync roots.
