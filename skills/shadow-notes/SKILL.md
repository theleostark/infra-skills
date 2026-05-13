---
name: SHADOW Notes
description: 'Use when the user needs: shadow-notes — SHADOW-ENT sticky note surface via Outlook Notes. Read, write, clear, sync todo/actions to work laptop. Part of Astemo enterprise plugin.'
icon: icon.svg
type: implementation
metadata:
  category: enterprise/astemo
  family: shadow
  lifecycle: active
  canonical_slug: astemo-shadow-notes
  icon_style: craft-category-glyph-v1
---

# shadow-notes

SHADOW-ENT product. Enterprise sticky note surface synced via M365 Outlook Notes.
Visible on: work laptop (Sticky Notes app), phone (Outlook), web (outlook.office365.com/mail/notes).

## Architecture

```
shadow-notes (skill) → tools_notes.py (CLI) → Playwright (Outlook Notes web)
                                              → M365 sync → Windows Sticky Notes
                                              → Outlook mobile
```

## Note Format (SD | Todo)

```
SD | Todo  ·  YYYY-MM-DD

P0 OPEN:
- [action]  Owner  Due

P1 WATCH:
- [item]

DONE:
- [completed items]

NEXT:
- [carry-forward]
```

Rules:
- Title line: `SD | Todo  ·  YYYY-MM-DD`
- Sections: P0 OPEN, P1 WATCH, DONE, NEXT
- One note only — overwrite each session (no accumulation)
- Color: Yellow (default)
- Content from: Bee actions, meeting notes, tracker, Smartsheet
- Auto-clear DONE section each new session
- Carry NEXT → P0/P1 on session start

## Operations

| Command | Action |
|---------|--------|
| `notes read` | Read current sticky note content |
| `notes write` | Overwrite with new SD Todo |
| `notes clear` | Delete all notes, create fresh |
| `notes push "text"` | Append a line to P0 or P1 |
| `notes sync` | Pull from Bee + tracker → generate note |
| `notes status` | Show note count and last modified |

## Module

`tools/shadow_enterprise_cli/tools_notes.py`

## URL

`https://outlook.office365.com/mail/notes`

## Workflow

### Session Start
1. Read current note
2. Move NEXT items → P0/P1
3. Clear DONE section
4. Add new items from Bee brief + tracker
5. Write updated note

### Session End
1. Compile DONE from session actions
2. Identify carry-forward → NEXT
3. Overwrite note
4. Note syncs to work laptop automatically

### Mid-Session Update
1. `notes push "FTA request sent to JP DE lead"` → appends to DONE
2. `notes push "P0: Confirm MQ ship decision"` → appends to P0

## Integration

- **Bee:** `owl_brief` → extract action items → P0/P1
- **Tracker:** open change points with no owner → P1
- **Smartsheet:** overdue actions → P0
- **Meeting prep:** upcoming meeting actions → P0
- **Teams:** unread from key contacts → P1

## Rules

1. One note only — never accumulate multiple notes
2. Overwrite pattern — each write replaces entire content
3. SD | Todo format — consistent structure every time
4. Yellow color — enterprise standard
5. Session-scoped DONE — clears each session start
6. NEXT carries forward — never lose items
7. Browser automation via Playwright — no API token needed
8. Syncs automatically via M365 — no manual push
