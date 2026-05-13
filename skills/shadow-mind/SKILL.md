---
name: SHADOW Mind
description: 'Use when the user needs: Context assembly, state computation, context health, experience capture, latent pattern detection. Use on "what am I working on", "context", "catch me up", "system state", "context health", "memory stats", "renew context", "what focus mode", "operator presence".'
icon: icon.svg
metadata:
  depends_on: [shadow-notes]
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-mind
  icon_style: craft-category-glyph-v1
---

# Shadow Mind

| Op | How |
|----|-----|
| **Context Assembly** | `esp.assemble` — git, mesh, memory, tabs, Bee, cloud |
| **State** | Focus detection, synthesized sensors, operator presence, mesh health |
| **Health** | Memory count, skill inventory, stale detection, disk per category |
| **Experience** | Write discoveries + patterns to memory for future sessions |
| **Latent Patterns** | SP-006 PostToolUse hook monitors tool sequences, flags for crystallization |

## Pipeline

```
Input (trigger phrase or automatic)
  ↓
Context Assembly (esp.assemble → git, mesh, memory, tabs, Bee, cloud)
  ↓
State Computation (focus mode, operator presence, mesh health, disk pressure)
  ↓
Health Scoring (memory count, skill freshness, stale detection, per-category disk)
  ↓
Synthesis & Output
  ├── Inline compact summary (default)
  ├── ShadowArchive/80-reports/context-health-YYYY-MM-DD.md  (full report)
  ├── Experience write → memory store  (pattern capture)
  └── SP-006 latent pattern flag → crystallize proposal
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Compact status card: focus mode, mesh node count, memory stats, top stale items, disk pressure | Any context trigger |
| `brief` | One-line status: focus mode + mesh health + disk % | Quick pulse check |
| `full` | Full context assembly: all sensors, all nodes, memory inventory, skill audit, stale report, experience delta | `catch me up`, `system state`, explicit request |
| `raw` | Raw `esp.assemble` output, unprocessed | Debugging context assembly |
| `renew` | Force-refresh context window from sources, discard cached state | `renew context`, stale session |

## Artifact Routing

| Artifact | Path | Retention |
|----------|------|----------|
| Context health report | `ShadowArchive/80-reports/context-health-YYYY-MM-DD.md` | 90 days |
| Experience entries | Memory store (via `arc.store` or `shadow-notes`) | Permanent |
| Latent pattern flags | `ShadowArchive/80-reports/latent-patterns-YYYY-MM-DD.md` | Until crystallized |
| Stale skill report | `ShadowArchive/80-reports/skill-staleness-YYYY-MM-DD.md` | 30 days |

## Fallback Chain

1. **Primary:** `esp.assemble` + `arc` memory + live mesh sensors (`den.environment`, `ant.mesh`)
2. **Mesh unavailable:** Fall back to local-only context: git status, local memory files, disk stats via `df`/`du`, focus mode detection from `~/Library/DoNotDisturb/DB/`
3. **ShadowArchive unmounted:** Use in-memory context only; note volume offline in output; skip disk audit for external volumes
4. **Bee unavailable:** Skip Bee facts/todos; note in output
5. **Last resort:** Bare `ls` + `git status` + `df -h /`; explicitly state degraded mode

## Prerequisites

- `esp.assemble` available (shadow-mcp or local shim)
- `arc` memory store accessible
- ShadowArchive volume mounted for full disk audit
- `den.environment` for hardware sensors (optional, graceful skip)
- `ant.mesh` for fleet topology (optional, graceful skip)
- Focus mode detection: `~/Library/DoNotDisturb/DB/` readable (macOS)

## Error Handling

| Failure | Recovery |
|---------|----------|
| `esp.assemble` timeout | Fall back to local git + disk stats; note degraded mode |
| Memory store corrupt/unreadable | Report count as 0, flag for repair; do not attempt auto-fix |
| Mesh node unreachable | Report node as offline; continue with reachable nodes |
| Volume unmounted | Skip disk audit for that volume; report "volume offline" |
| Stale detection finds >50% stale | Flag in output but do not auto-update; propose review |
| SP-006 hook crash | Log to `ShadowArchive/80-reports/sp006-error-*.md`; continue without latent detection |

## Contract

- **Boundaries:** shadow-mind is read-only for context assembly. It reports state; it does not modify mesh topology, move files, or change configuration.
- **Experience writes** are append-only to the memory store. shadow-mind never deletes or overwrites existing memories.
- **No auto-deletion:** Never prune memory, skills, or files based on staleness scores. Propose only.
- **No credential exposure:** Context reports must not contain API keys, tokens, or secrets. Redact before writing artifacts.
- **Externalization rule:** All context health reports go to `ShadowArchive/80-reports/`. Never leave the only copy in `/tmp`.
- **Do not** trigger mesh dispatch, offload, or tiering operations from shadow-mind. Those are separate skills with their own safety checks.
- **Operator presence** detection is local-only. Never phone home or report presence externally.

## Pipeline

```
Query → Gather context surfaces → Compute state → Present summary
```

1. Resolve operator query (what am I working on, catch me up, system state)
2. Gather from: focus mode, calendar, recent files, session history, git status
3. Compute current context zone and active work
4. Present structured summary

## Modes

| Mode | Output | When |
|------|--------|------|
| `catch-up` (default) | Full context summary | "catch me up", "what am I working on" |
| `context` | Zone + active surface only | "what context" |
| `health` | System health overview | "context health" |

## Artifact Routing

- Context summaries: chat only (ephemeral by nature)
- If operator requests durable capture: `ShadowArchive/80-reports/context-YYYY-MM-DD.md`

## Coverage Expansion (Cambrian Priority #2)

Current coverage: **~40%** of cross-session knowledge recall.
Target: **70%** by end of Q2 2026.

### Cross-Session Recall Pipeline

When operator asks "what did we decide about X?" or "where did we leave off on Y?":

```
1. Search session anchors in ShadowArchive/80-reports/
   → grep for topic keywords across *-session-anchor-*.md files
2. Search kernel snapshots in .shadow-kernel/snapshots/
   → JSON search for topic in summary fields
3. Search training pairs in .shadow-kernel/training-pairs-cambrian-*.jsonl
   → Relevance match on input fields
4. Search meeting notes in Dropbox/<program>/07_Meeting_Notes/
   → Full-text search across program meeting history
5. Synthesize: "Last session (DATE), we decided X. Evidence: [path]. Open items: [items]."
```

### Memory Write-Back Protocol

After every session that produces decisions or discoveries:

1. Write session anchor to `ShadowArchive/80-reports/`
2. Append key decisions to `.shadow-kernel/esp-context.md` under topic heading
3. If pattern discovered → crystallize to skill
4. If program-specific → write to `<program>/90_Astemo_Config/session-log.jsonl`

### Recall Scoring

| Confidence Level | Meaning | Source |
|---|---|---|
| **Confirmed** | Found in durable artifact (anchor, kernel, Dropbox) | File match |
| **Likely** | Found in training pairs or distilled experience | Pattern match |
| **Inferred** | Reconstructed from context but no explicit artifact | Reasoning |
| **Unknown** | No evidence found | No match |

Always label recall confidence. Never present inferred as confirmed.

### Unsolved Gaps

1. ❌ Semantic search over all session anchors (currently grep-only)
2. ❌ Auto-linking: "this decision supersedes the one from DATE" tracking
3. ⚠️ Decision decay: flagging when a decision from >30 days ago hasn't been acted on
4. ⚠️ Cross-program knowledge transfer (FH4S learning → FH4/FH4L application)
5. ❌ Temporal reasoning: "before the MLC meeting, we thought X; after, we decided Y"

## Contract

- Context is session-scoped — do not persist without operator request
- Respect zone boundaries — enterprise context does not leak into personal zone
- Recall confidence must always be labeled
- Inferred recall is never presented as confirmed
- Memory writes are append-only, never destructive
