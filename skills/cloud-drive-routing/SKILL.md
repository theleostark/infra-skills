---
name: Cloud Drive Routing
description: Map and route the operator's cloud, local drive, and subscribed storage surfaces across Dropbox, OneDrive, SharePoint/M365, Google Drive, iCloud, and SHADOW volumes. Use when asked to inventory cloud drives, choose where to save artifacts, resolve Dropbox/OneDrive/SharePoint/iCloud storage roles, check subscribed storage options, or create a Shadow-drive map.
icon: icon.svg
metadata:
  category: storage/cloud
  family: storage
  lifecycle: active
  canonical_slug: cloud-drive-routing
  icon_style: craft-category-glyph-v1
---

# Cloud Drive Routing

Use this skill to map what storage exists, what is already in use, and where a new artifact should live.

## Default routing

| Surface | Default role |
|---|---|
| Dropbox | Compact kernel, durable context, Astemo final roots, relay/control-plane notes |
| M365 SharePoint / OneDrive Business | Enterprise source-of-truth documents, team files, heavy work artifacts |
| OneDrive Personal | Personal files and large non-enterprise binaries after capacity confirmation |
| Google Drive | Google-native docs/sheets and shared personal files |
| iCloud Drive | Apple continuity and small cross-device personal relay, not bulk archive |
| `/Volumes/☯Duality` / `/Volumes/SHADOW` | Local agent workspace, ShadowArchive, warm/cold archive, offload staging |

## Inventory workflow

1. Check local cloud roots:
   - `~/Library/CloudStorage/Dropbox/`
   - `~/Library/CloudStorage/OneDrive-*`
   - `~/Library/CloudStorage/GoogleDrive-*`
   - `~/Library/Mobile Documents/com~apple~CloudDocs/`
2. Check mounted volumes with `df -h` and `/Volumes`.
3. Check installed/running clients without printing secrets:
   - Dropbox.app, OneDrive.app, Google Drive.app
   - launch agents/processes for Dropbox, OneDrive, Google Drive, iCloud `bird`
4. Search existing routing docs before inventing a new model.
5. Classify each surface by: subscription confidence, local materialization, enterprise/personal boundary, and best use.

## Storage decision rules

- If it is an Astemo/customer-facing final, prefer the confirmed Dropbox program root or approved M365/SharePoint team location.
- If it is compact text knowledge that agents should reuse, prefer Dropbox with pointers to heavy evidence.
- If it is large binary evidence, prefer OneDrive/M365/SharePoint and store a pointer in Dropbox.
- If it is local runtime/tooling, keep it in the repo or SHADOW/Duality; do not push implementation internals into Dropbox-visible artifacts.
- If plan/capacity is unverified, mark subscription confidence instead of guessing.

## Report shape

```md
# Shadow-drive Map

| Surface | Evidence | Path/access | Subscription confidence | Best use |
|---|---|---|---|---|

## Recommended routing
## Cleanup/follow-up
## Verification and residual gaps
```

## Pipeline

```
Input (trigger: where to save, cloud drive, storage inventory, which drive)
  ↓
Inventory Local Cloud Roots:
  ├── ~/Library/CloudStorage/Dropbox/
  ├── ~/Library/CloudStorage/OneDrive-*/
  ├── ~/Library/CloudStorage/GoogleDrive-*/
  └── ~/Library/Mobile Documents/com~apple~CloudDocs/
  ↓
Check Mounted Volumes (df -h, /Volumes)
  ↓
Check Running Clients (Dropbox.app, OneDrive.app, Google Drive.app, bird)
  ↓
Classify Surfaces (subscription, materialization, enterprise/personal, best use)
  ↓
Apply Routing Rules (Astemo final → Dropbox, large binary → OneDrive, agent workspace → SHADOW)
  ↓
Artifact Generation
  └── ShadowArchive/80-reports/cloud-drive-map-YYYY-MM-DD.md
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Full cloud drive inventory + routing recommendation | Any cloud drive trigger |
| `inventory` | List all surfaces with paths, roles, and capacity | `inventory drives`, `list drives` |
| `route <artifact>` | Recommend where a specific artifact should live | `where should I save X`, `route this file` |
| `map` | Generate Shadow-drive map report | `drive map`, `cloud map` |
| `capacity` | Check capacity across all surfaces | `drive capacity`, `how much space` |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Cloud drive map | `ShadowArchive/80-reports/cloud-drive-map-YYYY-MM-DD.md` | Full inventory + routing recommendations |
| Routing decision | Inline (chat) | Per-artifact routing advice |

## Fallback Chain

1. **Primary:** Check `~/Library/CloudStorage/` roots + `/Volumes` + running clients → classify → route
2. **Cloud client not running:** Report which clients are inactive; note that sync state is unknown
3. **Dropbox not synced:** Check `~/Library/CloudStorage/Dropbox/` for stale files; report sync lag; suggest `dropbox.py start`
4. **OneDrive not configured:** Report; skip OneDrive in routing; note gap
5. **Volume unmounted:** Report which volumes are offline; route to available surfaces only; note offline options
6. **Last resort:** Report available local paths only; explicitly note that cloud surfaces were not checked

## Prerequisites

- macOS with CloudStorage framework (Dropbox, OneDrive, Google Drive integration)
- iCloud Drive enabled (`~/Library/Mobile Documents/`)
- `df`, `du`, `ls` available for capacity checks
- Cloud sync clients installed (optional, for sync state)
- Access to `ShadowArchive/80-reports/` for map artifacts

## Error Handling

| Failure | Recovery |
|---------|----------|
| CloudStorage path not found | Report; client may not be installed; skip that surface |
| Permission denied on cloud root | Report; skip; do not attempt to fix permissions |
| Sync client not running | Report; suggest starting client; note sync state unknown |
| Volume offline | Report; route to available surfaces; note offline option for later |
| Ambiguous enterprise/personal classification | Ask operator; do not guess; routing to wrong boundary is worse than asking |

## Contract

- **Routing is advisory.** cloud-drive-routing recommends; it does not auto-move files or change sync settings.
- **Enterprise/personal boundary is strict.** Never route personal/government/finance files to enterprise surfaces (OneDrive Business, SharePoint). Never route Astemo work to personal surfaces.
- **No credential exposure.** Drive maps must not contain auth tokens, sync keys, or share links with embedded credentials.
- **Subscription confidence is honest.** If plan/capacity is unverified, say so. Do not guess at storage limits.
- **Externalization rule.** Drive maps go to `ShadowArchive/80-reports/`. Never leave the only drive map in chat.
- **Do not** modify sync settings, share links, or client configurations. This skill is read-only inventory + routing advice.
- **Do not** route to surfaces that are at or near capacity.
- **Do not** assume cloud paths exist without checking. Always verify.

## Crystallized From

- Session: 2026-05-02 cloud-drive map and skill R&D pass.
- Pattern: storage questions need a bounded inventory and routing model across subscribed cloud surfaces plus SHADOW volumes.
