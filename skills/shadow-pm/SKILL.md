---
name: SHADOW Pm
description: 'Use when the user needs: Unified project management across Linear, Notion, Smartsheet, GitHub, Google Sheets, and Upscaler. Track work items, agent tasks, patent filing, sprint planning, mesh operations. Trigger on "track", "create issue", "project status", "what''s pending", "backlog", "sprint", "kanban", "assign", "priority", "deadline", "roadmap", "work items".'
icon: icon.svg
metadata:
  depends_on: [astemo-smartsheet,github-cli-token-ops,linear-graphql-api]
  category: enterprise/fh4s
  family: shadow
  lifecycle: active
  canonical_slug: shadow-pm
  icon_style: craft-category-glyph-v1
---

# shadow-pm — Unified Project Management

Orchestrate work across 6 PM surfaces from one skill.

## PM Surface Map

| Surface | Access | Best For |
|---------|--------|----------|
| **Linear** | MCP (mcp.linear.app) | Issues, sprints, kanban, agent tasks |
| **Notion** | MCP (plugin) | Docs, wikis, databases, knowledge base |
| **Smartsheet** | MCP (FRIDAY gateway) | Gantt charts, Astemo schedules, timelines |
| **GitHub** | `gh` CLI | PRs, code reviews, repo issues |
| **Google Sheets** | AACP workflow | Access control grants, approval queues |
| **Upscaler** | Custom API | Engineering programs (FH4S, XP7G) |

## Quick Commands

### Linear (via MCP or GraphQL)
```bash
# Create issue
# Use linear-graphql-api skill for programmatic access
# Or use Linear MCP tools: linear_create_issue, linear_search

# View active sprint
# Linear MCP: linear_get_active_cycle
```

### Notion (via MCP)
```bash
# Search across workspaces
# notion-search query="shadow-verse"

# Create a page
# notion-create-pages with parent and properties
```

### GitHub (via gh CLI)
```bash
gh issue list --repo ishuru/shadow-lab
gh issue create --title "SP-003 draft" --body "..."
gh pr list
```

### Smartsheet (via FRIDAY gateway)
```bash
# Uses keychain token: service=smartsheet, account=api-token
# smartsheet_list_sheets, smartsheet_get_sheet, smartsheet_add_rows
```

### Google Sheets (AACP)
```bash
python3 /Volumes/SHADOW/shadow-lab/apps/shadow-mcp/access-control/sheets_workflow.py list --status pending
```

## Work Item Routing

| Work Type | Route To | Why |
|-----------|----------|-----|
| Bug/feature | Linear | Kanban + sprint tracking |
| Patent filing | Linear + Notion | Track + document |
| Astemo schedule | Smartsheet | Gantt chart |
| Code PR | GitHub | Review workflow |
| Agent access | Google Sheets | AACP approval |
| Engineering program | Upscaler | Multi-program tracking |
| Documentation | Notion | Wiki + database |
| Mesh operation | shadow-top | TUI dashboard |

## Agent Task Tracking

For parallel sub-agents (like the 5 research agents running now):

```
Task: [agent-id] [description] [status]
─────────────────────────────────────────
a8cf35  CLI→MCP bridge research     RUNNING
a2c9bd  TUI framework research      RUNNING
a1d83a  MCP composition research    RUNNING
a2f145  Latent skills research      RUNNING
a63203  Skill audit                 RUNNING
```

Use Codex's TaskCreate/TaskUpdate for in-session tracking.
Use Linear for cross-session persistence.

## Shadow-Verse Roadmap

Track via Linear project or Notion database:

| Priority | Item | Status | Patent |
|----------|------|--------|--------|
| P0 | shadow-harness (CLI→MCP bridge) | Designing | SP-007 |
| P0 | shadowlab.cc redesign + deploy | Queued | — |
| P1 | shadow-powers private repo | Planned | — |
| P1 | SP-003 patent draft | Blocked (qwen3 pipeline) | SP-003 |
| P1 | TRMNL WiFi setup | Blocked (2.4GHz) | — |
| P2 | Svelte AACP dashboard | Planned | — |
| P2 | shadow-top mesh panels | Planned | — |
| P2 | dexo recovery | Lost anchor | — |
| P3 | shadow-eye (Frigate NVR) | Concept | — |
| P3 | shadow-flora (plant IoT) | Concept | — |

## Pipeline

```
Input (trigger: track, create issue, project status, backlog, sprint, assign, priority, deadline)
  ↓
Classify Work Type (bug, feature, patent, schedule, PR, agent task, doc, mesh op)
  ↓
Route to PM Surface:
  ├── Linear → issues, sprints, kanban
  ├── Notion → docs, wikis, databases
  ├── Smartsheet → Gantt, Astemo schedules
  ├── GitHub → PRs, code reviews, repo issues
  ├── Google Sheets → AACP access grants
  └── Upscaler → multi-program engineering tracking
  ↓
Execute (create/update/query via MCP or CLI)
  ↓
Artifact Generation
  ├── ShadowArchive/80-reports/pm-status-YYYY-MM-DD.md  (project rollup)
  └── Cross-surface sync (issue created → Notion page, etc.)
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Auto-route to the best PM surface based on work type | Any PM trigger |
| `status` | Cross-surface project status rollup | `project status`, `what's pending` |
| `sprint` | Active sprint view from Linear | `sprint`, `kanban` |
| `backlog` | Unprioritized items across surfaces | `backlog` |
| `create` | Create a new work item on the appropriate surface | `create issue`, `track` |
| `assign` | Assign ownership and priority | `assign`, `priority` |
| `roadmap` | Timeline view across programs | `roadmap`, `deadline` |
| `sync` | Cross-surface synchronization (e.g., Linear ↔ Notion) | `sync issues` |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Project status rollup | `ShadowArchive/80-reports/pm-status-YYYY-MM-DD.md` | Cross-surface project health |
| Sprint report | `ShadowArchive/80-reports/sprint-report-YYYY-MM-DD.md` | Sprint metrics and burndown |
| Roadmap snapshot | `ShadowArchive/80-reports/roadmap-YYYY-MM-DD.md` | Timeline and milestone tracking |

## Fallback Chain

1. **Primary:** MCP tools for Linear/Notion/Smartsheet → `gh` CLI for GitHub → API for Upscaler
2. **Linear MCP unavailable:** Fall back to `linear-graphql-api` skill (direct GraphQL API calls)
3. **Smartsheet MCP unavailable:** Fall back to `astemo-smartsheet` skill (direct API with keychain token)
4. **GitHub `gh` not authenticated:** Fall back to `github-cli-token-ops` skill for multi-account token management
5. **Notion MCP unavailable:** Report; use Linear as primary tracker; note Notion sync pending
6. **All MCP surfaces down:** Store work item locally in `ShadowArchive/80-reports/pm-queue-YYYY-MM-DD.json`; sync when surfaces recover

## Prerequisites

- Linear MCP connected (or `linear-graphql-api` skill + API key)
- Notion MCP connected (optional, graceful skip)
- `gh` CLI authenticated for GitHub operations
- Smartsheet API token in macOS Keychain (`service=smartsheet, account=api-token`) for schedule work
- Upscaler API access for engineering program tracking
- AACP Google Sheets workflow for access control grants

## Error Handling

| Failure | Recovery |
|---------|----------|
| Linear API rate limited | Back off 2s; retry once; report if still blocked |
| Smartsheet token expired | Report; suggest keychain update via `astemo-smartsheet` skill |
| `gh` auth failure | Route to `github-cli-token-ops` for token refresh |
| Work type ambiguous | List candidate surfaces; ask operator; do not silently pick one |
| Cross-surface sync conflict | Report both versions; ask operator to resolve; do not auto-merge |
| Sprint/cycle not found in Linear | List available cycles; ask operator to select or create |

## Contract

- **Route, don't duplicate.** Each work item should live on ONE primary surface. Cross-surface links are references, not copies.
- **No auto-prioritization.** Priority and deadline changes require operator approval. shadow-pm proposes; it does not auto-reprioritize.
- **No credential exposure.** PM reports must not contain API tokens or auth headers. Redact before writing artifacts.
- **Externalization rule.** All project status reports go to `ShadowArchive/80-reports/`. Never leave the only project state in chat.
- **Do not** modify Smartsheet schedules without operator approval (those are customer-visible).
- **Do not** create Linear issues on behalf of other team members without attribution.
- **Do not** auto-close issues based on PR merge unless the operator's workflow explicitly requires it.
