---
name: Bitwarden
description: Use when accessing bitwarden vault — search, extract credentials, manage entries. Uses `bw` CLI (invisible, no browser). Trigger on "bitwarden", "vault", "password lookup", "credential", "find password", "check bitwarden", "bw".
icon: icon.svg
metadata:
  category: automation/cli
  family: automation
  lifecycle: active
  canonical_slug: bitwarden-browser
  icon_style: craft-category-glyph-v1
  replaces_browser: true
  invisible_first: true
  tier: 1
---

# Bitwarden — Invisible CLI Access

Access the Bitwarden vault via `bw` CLI. Zero browser. Zero focus theft.

## Why CLI, Not Browser

`bitwarden-browser` used agent-browser to navigate vault.bitwarden.com — slow (10+ seconds), requires 2FA every session, and steals focus. The `bw` CLI is instant, scriptable, and invisible.

## Authentication

```bash
# Check if vault is unlocked
bw unlock --check
# → Vault is locked.

# Unlock vault — outputs session key
bw unlock --raw
# → <session_key>

# Set session for subsequent commands
export BW_SESSION="<session_key>"

# Or pass session inline
bw --session "<session_key>" list items
```

**Important:** Master password must come from the operator. Never store it in skills or memory.

## Common Operations

```bash
# Search vault
bw list items --search "github"
bw list items --search "astemo"

# Get specific item
bw get item "GitHub Token"
bw get item "sabarish.duvvuru@gmail.com"

# Get just the password
bw get password "GitHub Token"

# Get username
bw get username "GitHub Token"

# Get TOTP code
bw get totp "GitHub Token"

# Get full item as JSON
bw get item "GitHub Token" --pretty

# List all items (table view)
bw list items --fields name,login.username,login.uris

# Get URI for an item
bw get uri "GitHub Token"
```

## Sync

```bash
# Sync vault (pull latest changes)
bw sync

# Check sync status
bw sync --last
```

## Security Rules

1. **Never store the master password** — always ask operator
2. **Never cache BW_SESSION beyond the current command** — pipe it
3. **Never log credential values** — log item names only
4. **Redact in output** — mask passwords unless operator explicitly asks
5. **Delete session key after use** — unset BW_SESSION when done

## Invisible-First Pattern

```bash
# One-liner: unlock → get password → use → done
BW_SESSION=$(bw unlock --passwordenv BW_MASTER --raw) \
  bw --session "$BW_SESSION" get password "API Key" | pbcopy
```

No browser. No focus change. No screenshots. ~1 second.

## Pipeline

```
Intent → bw unlock (if needed) → bw get/list → Return structured output → Clear session
```

1. Check `bw unlock --check`
2. If locked, ask operator for master password
3. Unlock, capture session key
4. Run query (`bw get`, `bw list`, `bw search`)
5. Return result
6. Unset `BW_SESSION`

## Vault Accounts

| Email | Vault |
|-------|-------|
| sabarish.duvvuru@gmail.com | Primary personal vault |
| clawd.ctrl@gmail.com | Secondary vault |
| sabarish.duvvuru@outlook.com | Microsoft vault |
| theleostark@outlook.com | Legacy vault |

## Error Handling

| Error | Fix |
|-------|-----|
| `Vault is locked` | Ask operator for master password, run `bw unlock --raw` |
| `Session key expired` | Re-unlock |
| `Item not found` | Try `bw list items --search` with broader terms |
| `bw not found` | Install: `npm install -g @bitwarden/cli` |

## Contract

- **Never enter credentials** into browsers
- **Never submit forms** without operator approval
- **CLI-first always** — no browser automation for Bitwarden
- **Session key is ephemeral** — never persisted to disk
