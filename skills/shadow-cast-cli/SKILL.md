---
name: SHADOW Cast CLI
description: Use when rendering shadow-cast context primitives to CLI surfaces (Codex, Gemini CLI, Codex CLI). Cast context as ANSI-colored panels, tables, trees, and badges. Use when rendering mesh status, session anchors, fleet dashboards, or any structured context output in terminal.
icon: icon.svg
user-invocable: true
metadata:
  depends_on: [shadowtech-design-system]
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-cast-cli
  icon_style: craft-category-glyph-v1
---

# shadow-cast-cli — CLI Surface Adapter

Load `shadow-cast` for primitive schemas. Load `shadowtech-design-system` Terminal Context section for ANSI tokens.

## Surface Selection

| Context | Surface | Method |
|---------|---------|--------|
| Cowork mode (orchestration) | ANSI | `printf` in Bash tool |
| Chat mode (conversation) | Markdown | Direct text output |
| Code mode (implementation) | Inline markdown | Minimal, don't interrupt |

## Mode Detection

- **code**: Last messages involved file edits, test runs, debugging
- **chat**: Conversational Q&A, explanations, brainstorming
- **cowork**: `/smt`, fleet checks, session anchoring, dispatch, skill creation

Override: user says "compact" → code mode, "full" → cowork mode.

## ANSI Codes Reference

```
\033[0m  reset    \033[1m  bold     \033[2m  dim
\033[32m green    \033[33m yellow   \033[31m red
\033[35m magenta  \033[94m brt blue \033[95m brt mag
\033[90m dk gray
```

---

## 1. Status Panel

**Primitive:** N × Signal in bordered box. Groups by `source` for mesh signals.

### Cowork (ANSI)
```bash
printf "\033[1m╔═══ %s ═══════════════════════════════════╗\033[0m\n" "Title"
printf "║ \033[32m●\033[0m %-8s ✅ %-28s ║\n" "API" "OK — v2.3.0"
printf "║ \033[33m●\033[0m %-8s ⚠  %-28s ║\n" "Disk" "74% (62 GB free)"
printf "║ \033[31m○\033[0m %-8s ❌ %-28s ║\n" "ULTRON" "offline (12h)"
printf "\033[1m╚═══════════════════════════════════════════╝\033[0m\n"
```

### Chat (markdown)
```
─── Title ────────────────
● API:OK  ● Mesh:6/8  ⚠ Disk:74%  ❌ ULTRON
```

### Code (inline)
```
> **Status** ● API:OK ● Mesh:6/8 ⚠ Disk:74%
```

---

## 2. KPI Strip

**Primitive:** N × Signal with numeric values + progress bars.

### Cowork (ANSI)
```bash
printf "\033[1m┌─ Fleet KPIs ──────────────────────────────┐\033[0m\n"
printf "│ Disk    \033[33m████████████░░░░░░\033[0m  74%%  62 GB    │\n"
printf "│ Nodes   \033[32m██████░░░░░░░░░░░░\033[0m  6/8  online   │\n"
printf "│ Uptime  \033[32m████████████████░░\033[0m  94%%  11 days  │\n"
printf "\033[1m└───────────────────────────────────────────┘\033[0m\n"
```

### Chat
```
Disk ████████████░░░░ 74% │ Nodes 6/8 │ Uptime 94%
```

---

## 3. Table

**Primitive:** N × Signal as tabular rows with header.

### Cowork (ANSI)
```bash
printf "\033[1m┌──────────┬───────────┬────────┬─────────┐\033[0m\n"
printf "\033[1m│ Device   │ Status    │ Role   │ Uptime  │\033[0m\n"
printf "\033[1m├──────────┼───────────┼────────┼─────────┤\033[0m\n"
printf "│ JARVIS   │ \033[32m● online\033[0m  │ dev    │ 6h      │\n"
printf "│ AURION   │ \033[32m● online\033[0m  │ server │ 11d     │\n"
printf "│ ULTRON   │ \033[31m○ offline\033[0m │ desk   │ —       │\n"
printf "\033[1m└──────────┴───────────┴────────┴─────────┘\033[0m\n"
```

### Chat (markdown)
```markdown
| Device | Status | Role | Uptime |
|--------|--------|------|--------|
| JARVIS | online | dev | 6h |
```

---

## 4. Header / Banner

**Primitive:** Label + optional timestamp + metadata.

### Cowork (ANSI)
```bash
printf "\033[1m\033[35m╔═══ %s ════════════════════════════════╗\033[0m\n" "Fleet Topology"
printf "\033[35m║\033[0m 2026-03-25 21:30 │ 6 nodes │ 9 services  \033[35m║\033[0m\n"
printf "\033[1m\033[35m╚═══════════════════════════════════════════╝\033[0m\n"
```

### Chat
```
═══ Fleet Topology ═══ 6 nodes │ 9 services
```

### Code
```
── Fleet ── 6/8
```

---

## 5. Tree

**Primitive:** Vector with nested Steps (children present).

### Cowork (ANSI)
```bash
printf "\033[1m┌─ Read Order ──────────────────────┐\033[0m\n"
printf "│ AGENTS.md                         │\n"
printf "│ \033[2m├──\033[0m ShadowArchive/README.md       │\n"
printf "│ \033[2m├──\033[0m ShadowArchive/AGENTS.md       │\n"
printf "│ \033[2m├──\033[0m 03-agent-roots/README.md      │\n"
printf "│ \033[2m└──\033[0m 03-agent-roots/codex/program  │\n"
printf "\033[1m└───────────────────────────────────┘\033[0m\n"
```

### Chat (markdown)
```markdown
- AGENTS.md
  - ShadowArchive/README.md
  - ShadowArchive/AGENTS.md
```

---

## 6. Progress Bar

**Primitive:** Single Signal with numeric value as gauge.

### Cowork (ANSI)
```bash
printf "Disk  \033[33m████████████████\033[90m░░░░░░░░\033[0m  74%%  [\033[2m62 GB free\033[0m]\n"
printf "Build \033[32m████████████████████████\033[0m  100%% ✅ done\n"
printf "Upload \033[94m██████████\033[90m░░░░░░░░░░░░░\033[0m  42%%  ⟶ 1.2 MB/s\n"
```

### Chat
```
**Disk:** `████████████████░░░░░░░░` 74% (62 GB free)
```

---

## 7. Alert / Callout

**Primitive:** Single Signal with state + message. Left accent bar.

### Cowork (ANSI)
```bash
printf "\033[33m┃\033[0m \033[33m⚠\033[0m  SHADOW disk at 74%% — approaching 80%% offload threshold.\n"
printf "\033[33m┃\033[0m    \033[2mConsider running shadowarchive-tiering before next session.\033[0m\n"
printf "\n"
printf "\033[31m┃\033[0m \033[31m❌\033[0m ULTRON offline 12h — check power/network if unintended.\n"
printf "\n"
printf "\033[32m┃\033[0m \033[32m✅\033[0m API healthy — v2.3.0, all 9 services active.\n"
```

### Chat (markdown)
```markdown
> **Warning:** SHADOW disk at 74% — approaching 80% threshold.
```

---

## 8. Badge Row

**Primitive:** N × Signal as inline status tags.

### Cowork (ANSI)
```bash
printf "[\033[32m✅ JARVIS\033[0m] [\033[32m✅ AURION\033[0m] [\033[32m✅ FRIDAY\033[0m] [\033[32m✅ OCI\033[0m] [\033[31m❌ ULTRON\033[0m] [\033[33m⚠ Pixel-2\033[0m]\n"
```

### Chat
```
`JARVIS:online` `AURION:online` `FRIDAY:online` `ULTRON:offline`
```

---

## Composite Rendering

**Anchor** = Header + Status Panel (its signals) + Tree (its vectors)

**Delta** = Header + Alerts (added as green, removed as red, changed as amber)

**SMT Brief** = Header("SMT Context") + Status Panel + Delta(from last anchor) + Alerts(degraded) + Badge Row(fleet) + Header("Recommended Action") + plain text

**Tensor** = inline tuple: `mesh-inference [p:0.8 c:0.6 n:5 u:0.74]`


## Pipeline

```
Intent → Resolve target → Execute operation → Verify result → Report
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Standard output | Normal use |
| `verbose` | Detailed diagnostics | Debugging |
| `json` | Machine-readable output | Programmatic use |

## Artifact Routing

- Reports: `ShadowArchive/80-reports/`
- Config changes: `system/controls/` or `config/system/`

## Prerequisites

- SHADOW infrastructure accessible
- Appropriate auth/SSH for mesh targets

## Contract

- Never modify production config without operator confirmation
- Report all changes with before/after diff
- Preserve existing configurations with backup

