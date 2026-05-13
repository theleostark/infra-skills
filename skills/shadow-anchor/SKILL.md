---
name: SHADOW Anchor
description: 'Use when the user needs: Auto-save session as a temporal anchor — summarize deliverables, decisions, queued items, generate kernel training pairs, compute delta from previous anchor, and feed the shadow-research pipeline. Implements SP-004. Trigger on "anchor session", "save session", "session summary", "what did we do", end of session, before /clear.'
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-anchor
  icon_style: craft-category-glyph-v1
---

# shadow-anchor — Temporal Session Anchoring

Capture the current session as a navigable temporal anchor. Each anchor is a **kernel snapshot** that feeds the shadow-research pipeline (Phase 1 crawler input).

## Architecture

**Hooks handle the automatic path** (registered in `~/.Codex/settings.json`):
- **PreCompact** → `shadow-anchor-hook.py precompact` — captures kernel hash + training pairs before compaction
- **PostCompact** → `shadow-anchor-hook.py postcompact` — captures delta (what changed during compaction)
- **Stop** → `shadow-anchor-hook.py stop` — captures session summary at session end

**This skill is the manual force-trigger** — use when:
- End of a productive session (before `/clear`)
- User asks "what did we do?" or "save this"
- Proactively after major milestones
- Mid-session checkpoint that needs a full memory file (hooks only write JSONL)

## When to Use This Skill vs Hooks

| Situation | What Fires | Output |
|-----------|-----------|--------|
| Context compresses (auto) | PreCompact + PostCompact hooks | JSONL entries in queue |
| Session ends (auto) | Stop hook | JSONL entry in queue |
| User says "anchor" / "save session" (manual) | **This skill** | Full memory file + JSONL + MEMORY.md update |
| Before `/clear` (manual) | **This skill** | Full memory file + JSONL + MEMORY.md update |

The hooks capture raw data. This skill creates the **human-readable session memory file** with structured sections.

## Procedure

### Step 0: Push to SHADOW-Agent-Tasks (cloud queue)
If browser is authenticated to `astemogroup.sharepoint.com`, create a task for this session's deliverables:
1. Cache digest via contextinfo
2. POST to `SHADOW-Agent-Tasks` with:
   - Title: session descriptor
   - Program: detected from context
   - TaskType: 'review' (if deliverable needs review) or 'ship' (if shipped)
   - Status: 'done'
   - Source: 'manual'
   - Output: summary of deliverables + file paths
   - Owner: 'Sabarish'

This creates a cloud-visible record even if the session context is cleared.

### Step 1: Scan Session State

Gather ALL of these (in parallel where possible):

```bash
git log --oneline -10
git diff --stat HEAD~N  # N = commits this session
ls -lt ~/.Codex/skills/*/SKILL.md | head -10
ls -lt ~/.Codex/projects/-Volumes-SHADOW/memory/session_*.md | head -3
git status --short
```

### Step 2: Load Previous Anchor

Read the most recent `session_*.md` from memory to compute the delta:
- What was queued last time?
- What got done from that queue?
- What's NEW that wasn't queued? (emergent work)

### Step 3: Generate Anchor Memory File

Write to `~/.Codex/projects/-Volumes-SHADOW/memory/session_YYYY_MM_DD_[tag].md`:

```markdown
---
name: Session YYYY-MM-DD [descriptor]
description: [one-line: what was built, key numbers]
type: project
---

## Completed
1. [deliverable with enough detail to verify]

## Commits
- `hash` message

## Memory Created/Updated
- [file] — [what and why]

## Key Decisions
- [decision]: [rationale] (not just WHAT, but WHY)

## Concepts Introduced
- **[name]**: [one-line definition and why it matters]

## Delta from Previous Session
- Completed from queue: [items from prior anchor's P0/P1]
- Emergent work: [items that weren't queued]
- Still queued: [items carried over]

## Queued for Next Session

### P0 (immediate)
- [item with full context for cold-start resume]

### P1
- [item]

### P2
- [item]
```

### Step 4: Generate Training Pairs

Append to `~/.shadow-research/queue/kernel-diff.jsonl`:

```json
{
  "id": "anchor-YYYY-MM-DD-tag",
  "timestamp": "ISO8601",
  "source_node": "hostname",
  "hook": "SessionAnchor",
  "captures": {
    "kernel_hash": "sha256:...",
    "facts_extracted": ["fact1", "fact2"],
    "decisions": [{"context": "...", "decision": "...", "reasoning": "..."}],
    "training_pairs": [{"q": "...", "a": "..."}],
    "concepts_introduced": [{"name": "...", "definition": "..."}]
  }
}
```

Or call the hook directly: `echo '<context>' | python3 ~/.Codex/hooks/shadow-anchor-hook.py stop`

### Step 5: Update MEMORY.md Index

Add a line under `## Session Logs`.

### Step 6: Announce

Output using shadow-cast-cli style: session theme, deliverable count, training pairs count, P0 queue.

## Pipeline

```
Input ("anchor" / "save session" / "what did we do" / end-of-session / pre-/clear)
  ↓
Session State Scan (git log, file changes, skills touched, memory files)
  ↓
Previous Anchor Load (compute delta: completed, emergent, carried-over)
  ↓
Anchor Generation (memory file + training pairs + MEMORY.md index)
  ↓
Artifact Routing
  ├── ~/.Codex/projects/-Volumes-SHADOW/memory/session_YYYY_MM_DD_[tag].md  (human-readable anchor)
  ├── ~/.shadow-research/queue/kernel-diff.jsonl                          (training pairs)
  ├── MEMORY.md index entry                                               (session log index)
  └── SharePoint SHADOW-Agent-Tasks (cloud queue, when authenticated)
```

## Modes

### Default: Full Anchor
- Complete session memory file with all sections: Completed, Commits, Decisions, Delta, Queue.
- Training pairs appended to kernel-diff queue.
- MEMORY.md index updated.

### brief
- Compact session summary: 5-line Completed list + P0 queue only. No training pairs.

### delta
- Only compute and output the delta from the previous anchor. No full memory file.

### queue
- Only output the P0/P1/P2 queue for next session. Skip other sections.

### dry-run
- Show what the anchor would contain without writing any files. Useful for mid-session checkpoint review.

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Session memory file | `~/.Codex/projects/-Volumes-SHADOW/memory/session_YYYY_MM_DD_[tag].md` | Human-readable navigable anchor |
| Training pairs | `~/.shadow-research/queue/kernel-diff.jsonl` | Research pipeline input |
| MEMORY.md index | `~/.Codex/projects/-Volumes-SHADOW/memory/MEMORY.md` | Session log index |
| Cloud task | SharePoint `SHADOW-Agent-Tasks` | Cloud-visible record when authenticated |

Do not leave session state only in chat output. Always persist at least the memory file.

## Fallback Chain

1. **Primary: Full anchor procedure** — Scan state → compute delta → write memory file + training pairs + index.
2. **Hooks-only capture** — When this skill is not invoked manually, PreCompact/PostCompact/Stop hooks capture JSONL entries automatically.
3. **Training pairs only** — When memory file directory is unavailable, append training pairs to kernel-diff queue as the minimum durable output.
4. **Chat-only summary** — When no filesystem access is available, output a compact session summary to chat with a note that it was not persisted.

When falling back: `⚠ Anchor not persisted to filesystem. Session summary is chat-only.`

## Prerequisites

- Codex/OpenCode workspace with memory directory (`~/.Codex/projects/-Volumes-SHADOW/memory/`)
- `git` in the current project (for commit/diff scanning)
- `shadow-anchor-hook.py` registered in settings (for hook-based automatic capture)
- SharePoint authentication (optional, for cloud queue)
- `~/.shadow-research/queue/` writable (for training pairs)

## Error Handling

| Failure | Recovery |
|---------|----------|
| Memory directory does not exist | Create it; proceed with anchor write |
| No previous anchor for delta | Skip delta section; note "first anchor" in output |
| git not available (not a repo) | Skip commit/diff sections; note git unavailable |
| Training pairs file not writable | Write to `/tmp/kernel-diff-YYYY-MM-DD.jsonl` as fallback; note alternate path |
| SharePoint not authenticated | Skip cloud queue push; note in anchor that cloud sync is pending |
| Session was purely conversational (no files changed) | Still write anchor; mark Completed as "discussion only" with key decisions |

## Contract

- **Absolute dates only:** Never use relative dates ("yesterday", "last week") in anchor files. Always use ISO dates.
- **Cold-start context:** Every P0 queue item must contain enough context for a fresh session to resume without chat history.
- **Decisions include rationale:** Record WHY, not just WHAT. A decision without reasoning is not useful for future sessions.
- **Delta is required:** Always reference the previous anchor and compute delta. New anchors without delta lose temporal continuity.
- **Hooks complement this skill:** Hooks (PreCompact/PostCompact/Stop) capture raw data automatically. This skill creates the human-readable anchor. Both should coexist.
- **SHADOW separation:** Keep SHADOW-internal references (patent IDs, kernel hashes, research queue paths) in SHADOW-local artifacts only. Do not expose in cloud queue items.
- **Idempotent:** Running anchor twice for the same session should produce the same memory file (or note that it's a re-anchor).
- **No session modification:** This skill observes and records. It does not change project files, git state, or session behavior.

## Quality Checks

- [ ] No relative dates — use absolute dates
- [ ] Every P0 has cold-start context
- [ ] Decisions include rationale
- [ ] Delta references previous anchor
- [ ] Training pairs are factual

