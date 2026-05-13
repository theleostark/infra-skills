---
name: Storage Ops
description: 'Use when the user needs: Storage management — audit roots, tier hot/warm/cold, offload to SHADOW, clean caches, generate manifests. Use on "disk space", "storage", "offload", "clean up", "free space", "audit storage", "tier", "move to SHADOW", "cache cleanup", "node_modules", or any storage operation.'
icon: icon.svg
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: storage-ops
  icon_style: craft-category-glyph-v1
---

# Storage Operations

| Op | What |
|----|------|
| **Root Audit** | Classify top-level items into canonical categories |
| **Tiering** | Apply hot/warm/cold model across SSD + SHADOW |
| **Offload** | Move large tree to SHADOW, replace with symlink |
| **Cache Clean** | Scan node_modules, .pio, __pycache__, .cache across mesh |
| **Manifest** | Generate migration manifest for planned moves |

Rules: Never auto-delete. Route ambiguous to `70-quarantine`. Keep MBA SSD lean. Consult `offload_rules.md`.

## Pipeline

```
Input (trigger phrase: disk space, storage, offload, clean up, tier, cache cleanup)
  ↓
Diagnose (df -h, du -sh major trees, identify pressure)
  ↓
Route to sub-operation:
  ├── Root Audit → classify root entries
  ├── Tiering → assign hot/warm/cold placement
  ├── Offload → copy to SHADOW + symlink
  ├── Cache Clean → scan + report node_modules, .pio, __pycache__, .cache
  └── Manifest → generate planned-moves manifest
  ↓
Execute sub-operation with safety checks
  ↓
Artifact Generation
  ├── ShadowArchive/80-reports/storage-ops-YYYY-MM-DD.md  (session report)
  ├── ShadowArchive/80-reports/offload-manifest-*.md        (offload audit trail)
  └── ShadowArchive/80-reports/cache-scan-*.md              (cache cleanup report)
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Auto-diagnose + route to the most needed sub-operation | Any storage trigger |
| `audit` | Root audit report for a given volume | `audit storage`, `audit root` |
| `tier` | Tiering plan with hot/warm/cold assignments | `tier storage`, `tiering` |
| `offload <path>` | Offload specific path to SHADOW | `offload <path>`, `move to SHADOW` |
| `cache` | Scan and report cache directories; propose cleanup | `cache cleanup`, `clean up` |
| `manifest` | Generate migration manifest without executing | `manifest`, `plan moves` |
| `status` | Quick disk usage summary across local + SHADOW volumes | `disk space`, `free space` |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Session report | `ShadowArchive/80-reports/storage-ops-YYYY-MM-DD.md` | What was diagnosed, routed, executed |
| Offload manifest | `ShadowArchive/80-reports/offload-manifest-<slug>-YYYY-MM-DD.md` | Audit trail per offload |
| Cache scan report | `ShadowArchive/80-reports/cache-scan-YYYY-MM-DD.md` | Found caches with sizes |
| Tiering plan | `ShadowArchive/80-reports/tiering-plan-YYYY-MM-DD.md` | Hot/warm/cold assignments |
| Migration manifest | `ShadowArchive/80-reports/migration-manifest-YYYY-MM-DD.md` | Planned moves before execution |

## Fallback Chain

1. **Primary:** Auto-diagnose → route to sub-operation → execute with safety checks
2. **SHADOW volume unmounted:** Operate on local disk only (audit, tiering plan, cache scan); note SHADOW offline; skip offload
3. **Sub-operation skill unavailable:** Execute equivalent logic inline with the same safety rules
4. **rsync unavailable:** Fall back to `cp -a` for copies
5. **Permission denied on some directories:** Report accessible trees; mark denied as `unknown-permission-denied`; do not `sudo`
6. **Last resort:** Diagnostic-only mode — report what should happen without executing any moves

## Prerequisites

- macOS or Linux with standard unix tools (`df`, `du`, `ls`, `file`, `ln`, `cp`)
- SHADOW/ShadowArchive volume mounted (required for offload; optional for audit/tier/cache)
- `rsync` preferred for copies (falls back to `cp -a`)
- Write access to `ShadowArchive/80-reports/` for artifacts
- `offload_rules.md` consulted when available (per-site rules)

## Error Handling

| Failure | Recovery |
|---------|----------|
| Volume unmounted | Report offline; proceed with available volumes |
| Disk full during copy | Abort that copy; do not symlink; report partial state |
| Symlink already exists at target | Report conflict; do not overwrite without approval |
| Cache directory in use (node_modules with running process) | Skip; report as in-use; do not force-remove |
| `du` times out on large tree | Sample top levels; estimate size; note incomplete |
| Ambiguous classification | Route to `70-quarantine`; do not guess canonical destination |

## Contract

- **Never auto-delete.** All delete operations require explicit operator confirmation. Default is to move or quarantine, not delete.
- **Route ambiguous to `70-quarantine`.** If classification is uncertain, quarantine rather than destroy.
- **Keep MBA SSD lean.** The primary goal is maintaining usable free space on the boot volume.
- **Consult `offload_rules.md`.** Site-specific rules take precedence over defaults.
- **Reversibility.** Every offload produces a rollback record. Moves without rollback paths are a bug.
- **Volume integrity.** Never write to a volume with <5% free space after the proposed operation.
- **Externalization rule.** All reports and manifests go to `ShadowArchive/80-reports/`. Never leave the only record in chat or `/tmp`.
- **Do not** touch Dropbox, iCloud, OneDrive, or Google Drive sync roots.
- **Do not** modify running process working directories.
- **Do not** execute moves from the cache scan. Cache scan reports only; cleanup requires separate approval.

## Pipeline

```
Root scan → Classify tiers → Generate manifest → Propose actions → Execute (approved)
```

1. Scan target root(s) for size, file count, age distribution
2. Classify into hot/warm/cold tiers
3. Generate storage manifest with recommendations
4. Present actions for operator approval
5. Execute only approved actions

## Modes

| Mode | Output | When |
|------|--------|------|
| `audit` (default) | Full storage report | "audit storage", "disk space" |
| `clean` | Propose cleanup actions | "clean up", "reclaim space" |
| `offload` | Move cold data to archive | "offload to SHADOW" |

## Artifact Routing

- Storage manifests: `ShadowArchive/80-reports/storage-manifest-YYYY-MM-DD.md`
- Cleanup reports: `ShadowArchive/80-reports/storage-cleanup-YYYY-MM-DD.md`

## Prerequisites

- Target volume/directory accessible
- Write permission for move/delete operations

## Error Handling

- **Volume unmounted**: Report and skip, do not wait
- **Permission denied**: Log and skip file, continue scan
- **Large tree**: Sample first 10000 files for quick estimate

## Contract

- **Never delete without operator approval** — always propose first
- **Never delete .git directories** — even if large
- **Preserve symlinks** — do not follow and delete symlink targets
- Report sizes honestly — do not round favorably
