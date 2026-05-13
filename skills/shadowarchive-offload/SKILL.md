---
name: 'ShadowArchive Offload'
description: 'Use when the user needs: Offload a large local tree onto SHADOW or ShadowArchive while preserving the original path via symlink. Use when reclaiming local SSD space from low-risk targets such as experimental trees, large corpuses, or inactive project assets.'
icon: icon.svg
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: experimental
  canonical_slug: shadowarchive-offload
  icon_style: craft-category-glyph-v1
  replaced_by: conservative
---

Offload a large local tree to SHADOW conservatively.

## Use when

- the source path is large and low-risk
- the destination volume is mounted and has capacity
- the original path should keep working after offload

## Workflow

1. Verify the source is not a dangerous active path.
2. Copy the tree to the canonical SHADOW destination.
3. Validate the copy with a dry-run compare.
4. Preserve a rollback copy when needed.
5. Replace the original path with a symlink to the SHADOW canonical copy.
6. Re-check path usability and free-space improvement.

## Preferred targets

- `_experimental/*`
- corpuses
- archived outputs
- large inactive assets

## Safety

- Avoid active build roots unless explicitly requested.
- Prefer conservative rollback paths over irreversible deletion.
- Verify the symlinked path resolves correctly before declaring success.

## Pipeline

```
Input (path to offload + destination volume)
  ↓
Safety Check (dangerous path? active build root? git worktree?)
  ↓
Copy Tree → SHADOW canonical destination
  ↓
Validation (dry-run compare, file count, byte count)
  ↓
Symlink Swap (original path → SHADOW copy)
  ↓
Post-verification (path resolves, disk space reclaimed)
  ↓
Artifact Generation
  ├── ShadowArchive/80-reports/offload-manifest-<path-slug>-YYYY-MM-DD.md
  └── Rollback record (original path, destination, timestamp)
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Offload single path: copy → validate → symlink → report | Explicit path given |
| `batch` | Process multiple paths from a manifest file | User provides list or audit report |
| `dry-run` | Show what would move, sizes, destinations — no actual copy | `--dry-run` or confirmation step |
| `estimate` | Report potential space savings without touching files | `estimate offload`, sizing question |
| `rollback` | Reverse last offload: restore original, remove symlink, delete SHADOW copy (with confirmation) | `rollback offload` |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Offload manifest | `ShadowArchive/80-reports/offload-manifest-<slug>-YYYY-MM-DD.md` | Audit trail of what moved where |
| Rollback record | `ShadowArchive/80-reports/offload-rollback-<slug>-YYYY-MM-DD.md` | Undo instructions |
| Validation log | `ShadowArchive/80-reports/offload-validate-<slug>-YYYY-MM-DD.log` | File count + byte count proof |

## Fallback Chain

1. **Primary:** Copy to SHADOW/ShadowArchive volume → validate → symlink
2. **Destination full:** Report capacity; propose alternative destination or partial offload; do not proceed with partial copy silently
3. **Destination unmounted:** Detect via `mount | grep`; report volume offline; ask operator to mount; do not attempt auto-mount
4. **Copy fails mid-transfer:** Leave partial copy in place; do NOT symlink; report failure with partial-copy path for manual cleanup
5. **Symlink fails (permissions):** Report error; keep both original and copy in place; do not delete original
6. **Last resort:** If SHADOW volume is unavailable and operator insists, offer tar.gz archival to a local backup location instead of offload

## Prerequisites

- SHADOW or ShadowArchive volume mounted and writable
- Sufficient destination capacity (check before copy with `df -h`)
- Source path not an active build root, running process working directory, or git worktree
- `rsync` or `cp -a` available for faithful copy (preserving permissions, timestamps, symlinks)
- `ln -s` available for symlink creation
- Write access to `ShadowArchive/80-reports/` for manifest

## Error Handling

| Failure | Recovery |
|---------|----------|
| Source path not found | Report; skip; do not create empty placeholder |
| Source is a symlink already | Resolve target; report chain; offload the real target, not the symlink |
| Disk full during copy | Abort copy; do NOT symlink; leave partial copy for manual review |
| Permission denied on source | Report; skip; do not `sudo` without explicit operator approval |
| Permission denied on destination | Report; suggest `chmod` or alternate destination; do not proceed |
| rsync binary missing | Fall back to `cp -a`; note degraded copy in manifest |
| Validation mismatch (file count/bytes differ) | Do NOT symlink; report mismatch; leave both copies for manual inspection |
| Symlink target already exists | Report conflict; do not overwrite without explicit approval |

## Contract

- **No auto-deletion.** The original is removed only after validation passes AND the operator confirms. Default behavior keeps the original.
- **No auto-dedupe.** Offload does not deduplicate; it copies faithfully.
- **Reversibility required.** Every offload must produce a rollback record. Offload without a rollback path is a bug.
- **Volume integrity.** Never write to a volume that reports <5% free space after the proposed copy.
- **Dangerous paths are non-negotiable.** Active build roots (`node_modules` in use, running Docker volumes, live databases), running process working directories, and `/System` paths are always rejected unless the operator explicitly overrides with full acknowledgment.
- **Externalization rule.** All manifests and rollback records go to `ShadowArchive/80-reports/`. Never leave the only audit trail in `/tmp`.
- **Do not** chain offload into tiering or root-audit. Those are separate skills. Offload does one thing: move a tree and symlink.
- **Do not** offload to NFS/SMB mounts without explicitly warning about symlink semantics across filesystems.

## Pipeline

```
Target tree → Size audit → Create SHADOW copy → Verify integrity → Replace with symlink → Verify symlink
```

1. Identify target tree for offload
2. Copy to SHADOW destination (preserving structure)
3. Verify file count and total size match
4. Replace original with symlink
5. Verify symlink resolves correctly

## Artifact Routing

- Offload manifests: `ShadowArchive/80-reports/offload-manifest-YYYY-MM-DD.md`

## Fallback Chain

1. Direct copy to SHADOW volume
2. rsync with checksum verification
3. Abort if verification fails — original stays intact

## Prerequisites

- SHADOW destination volume accessible and writable
- Sufficient space on destination
- Original location supports symlinks

## Contract

- **Never delete original until copy verified** — two-phase: copy → verify → symlink
- **Verify after symlink** — confirm the symlink resolves before claiming complete
- Report: original path, SHADOW path, file count, total size, verification result
