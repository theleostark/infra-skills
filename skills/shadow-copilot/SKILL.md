---
name: SHADOW Copilot
description: 'Use when the user needs: Microsoft Copilot ecosystem CLI — Copilot Studio agents, Work IQ, M365 usage reports, credit tracking, product landscape. Use when the user mentions "copilot", "copilot studio", "work iq", "copilot usage", "copilot credits", "direct line", "copilot landscape".'
icon: icon.svg
metadata:
  filePattern:
  - '**/shadow-copilot/**'
  - '**/shadow_copilot*'
  bashPattern:
  - copilot
  - shadow-copilot
  - work iq
  - copilot studio
  category: enterprise/fh4s
  family: shadow
  lifecycle: active
  canonical_slug: shadow-copilot
  icon_style: craft-category-glyph-v1
---

# shadow-copilot — Microsoft Copilot Ecosystem CLI

Located at `~/tools/shadow-copilot/shadow_copilot.py`.
Config at `~/.config/shadow-copilot/config.json`.
FH4S Studio Agent Spec: `/Volumes/☯Duality/ShadowArchive/80-reports/shadow-copilot-fh4s-studio-agent-spec-2026-04-26.md`.

## Quick Reference

```bash
# Show product landscape
python tools/shadow-copilot/shadow_copilot.py landscape

# M365 Copilot usage for last 30 days
python tools/shadow-copilot/shadow_copilot.py usage --period D30

# Agent/app catalog
python tools/shadow-copilot/shadow_copilot.py catalog

# Work IQ status (MCP servers, Graph endpoints)
python tools/shadow-copilot/shadow_copilot.py workiq

# Credit consumption rates
python tools/shadow-copilot/shadow_copilot.py credits

# Chat with Copilot Studio agent
python tools/shadow-copilot/shadow_copilot.py studio chat

# List agents
python tools/shadow-copilot/shadow_copilot.py studio list
```

## Copilot Ecosystem (7 Tiers)

| Tier | Products | CLI Access |
|------|----------|------------|
| Core | M365 Copilot, Chat, Pro | Graph API |
| Developer | GitHub Copilot, Agent, Coding Agent | `gh copilot` CLI |
| Security | Security Copilot | API plugins, MCP |
| Builder | Copilot Studio, Power Automate/Apps/BI | Direct Line, `pac` CLI |
| Infra | Copilot for Azure, Windows | AI Shell, MCP connectors |
| Business | Dynamics 365 (Sales, Service, Finance, Fabric) | Copilot SDK |
| Collab | Pages, Notebooks, Search | Graph API |

## Key APIs

- **Graph Usage**: `GET /v1.0/copilot/reports/getMicrosoft365CopilotUsageUserDetail(period='D30')`
- **Agent Catalog**: `GET /v1.0/copilot/admin/catalog/packages`
- **Direct Line**: `POST directline.botframework.com/v3/directline/conversations/{id}/activities`
- **Work IQ MCP**: Mail, Calendar, Teams servers (preview)

## Extensibility Mechanisms

1. **MCP** — universal connector (M365, Studio, Windows, GitHub, Security)
2. **API Plugins** — OpenAPI specs for M365 Copilot
3. **Graph Connectors** — sync external data into semantic index
4. **Direct Line 3.0** — programmatic agent interaction
5. **pac CLI** — Power Platform solution ALM
6. **gh copilot** — GitHub Copilot CLI

## Status (2026-04-26)

- ✅ CLI tool built and tested at `~/tools/shadow-copilot/shadow_copilot.py`
- ✅ Config template created at `~/.config/shadow-copilot/config.json`
- ✅ FH4S Studio Agent design spec written
- ❌ Azure AD app registration — NOT YET REQUESTED from Astemo IT
- ❌ Graph API access — NO TOKENS (pending app registration)
- ❌ Direct Line secret — NOT CONFIGURED
- ❌ Copilot Studio agent — NOT BUILT (pending credentials)

## Auth

Config at `~/.config/shadow-copilot/config.json`:
- `tenant_id` — Azure AD tenant
- `client_id` / `client_secret` — app registration
- `direct_line_secret` — Copilot Studio channel secret
- `copilot_studio_env` — Power Platform environment ID

## Credit Rates (Copilot Studio)

| Feature | Credits |
|---------|---------|
| Classic answer | 1 |
| Generative (RAG) | 2 |
| Agent action | 5 |
| Graph grounding | 10 |
| Agent flow (per 100) | 13 |
| Premium AI (per 10) | 100 |

Prepaid: 25,000 credits / $200/month. M365 Copilot users: zero-rated.

## Pipeline

```
Input (trigger: copilot, copilot studio, work iq, copilot usage, copilot credits)
  ↓
Route to Sub-Command:
  ├── landscape → product ecosystem overview
  ├── usage → M365 Copilot usage report (Graph API)
  ├── catalog → agent/app listing
  ├── workiq → Work IQ MCP status
  ├── credits → credit consumption rates
  ├── studio chat → interact with Copilot Studio agent
  └── studio list → list deployed agents
  ↓
Execute CLI command or API call
  ↓
Artifact Generation
  ├── ShadowArchive/80-reports/copilot-usage-YYYY-MM-DD.md
  └── ShadowArchive/80-reports/copilot-landscape-YYYY-MM-DD.md
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Copilot landscape overview + auth status | `copilot`, `copilot landscape` |
| `usage` | M365 Copilot usage report for specified period | `copilot usage` |
| `credits` | Credit consumption rates and remaining balance | `copilot credits` |
| `studio` | Copilot Studio agent interaction | `copilot studio chat`, `copilot studio list` |
| `workiq` | Work IQ MCP server status | `work iq` |
| `catalog` | Agent/app catalog from Graph API | `copilot catalog` |
| `status` | Auth/config readiness check | `copilot status` |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Usage report | `ShadowArchive/80-reports/copilot-usage-YYYY-MM-DD.md` | M365 Copilot adoption metrics |
| Landscape report | `ShadowArchive/80-reports/copilot-landscape-YYYY-MM-DD.md` | Ecosystem overview |
| Studio agent spec | `ShadowArchive/80-reports/shadow-copilot-fh4s-studio-agent-spec-*.md` | Agent design spec |

## Fallback Chain

1. **Primary:** `shadow_copilot.py` CLI → Graph API / Direct Line / Power Platform
2. **CLI tool not found:** Report; suggest install from `~/tools/shadow-copilot/`; fall back to direct `curl` Graph API calls
3. **No Azure AD app registration:** Report that Graph API access is blocked; use landscape/credit data only (no live usage)
4. **No Direct Line secret:** Studio chat unavailable; report; suggest config at `~/.config/shadow-copilot/config.json`
5. **MFA on Graph API:** Cannot bypass; report; ask operator to complete auth flow
6. **Last resort:** Static landscape data from this skill + credit rate table; explicitly note no live data

## Prerequisites

- `shadow_copilot.py` at `~/tools/shadow-copilot/shadow_copilot.py`
- Config at `~/.config/shadow-copilot/config.json` with tenant_id, client_id, client_secret
- Azure AD app registration (pending Astemo IT approval for Graph API access)
- Direct Line secret for Copilot Studio chat (not yet configured)
- Python with `requests` library

## Error Handling

| Failure | Recovery |
|---------|----------|
| CLI tool missing | Report path; suggest install; fall back to manual API calls |
| Config file missing | Report; suggest `~/.config/shadow-copilot/config.json` template; cannot authenticate |
| Graph API 401/403 | Report auth failure; check if app registration exists; suggest IT request |
| Direct Line 404 | Studio agent not deployed; report; suggest building agent first |
| Credit data stale | Note data freshness; timestamp in report; cannot get real-time without API access |
| `pac` CLI not available | Skip Power Platform ALM commands; report degraded mode |

## Contract

- **No auto-provisioning.** shadow-copilot reports and queries; it does not create Azure AD apps, deploy agents, or modify Power Platform without explicit operator approval.
- **Azure AD app registration is a gate.** Graph API access requires IT approval. Do not attempt to self-register.
- **No credential exposure.** Reports must not contain client_secret, Direct Line secrets, or tenant IDs in externalized artifacts.
- **Credit tracking is informational.** Do not auto-throttle or block agent actions based on credit consumption. Report only.
- **Externalization rule.** All usage and landscape reports go to `ShadowArchive/80-reports/`. Never leave the only Copilot data in chat.
- **Do not** build Copilot Studio agents without operator approval and IT app registration.
- **Do not** expose Copilot Studio Direct Line secrets in logs or reports.
- **Do not** assume Graph API access. Always check auth status before attempting API calls.
