---
name: 'AI-Fi Manager'
description: Use when managing aPI keys and secrets via ai-fi (mesh-backed varlock env plus local fallbacks). Store, retrieve, scan, and feed keys from shared mesh config, peers, and local .env files into the unified gateway environment. Use when managing secrets, API keys, environment variables, or when setting up new services.
icon: icon.svg
command: ai-fi
user_invocable: true
metadata:
  depends_on: [aacp,mesh-ops,multi-model-query,shadow-connect]
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: ai-fi-manager
  icon_style: craft-category-glyph-v1
---

# ai-fi Manager — Mesh-backed Secret Operations

## System
- ai-fi module: `/Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs`
- Shared mesh config root: `/Users/Shared/shadow-labs/mesh-config`
- Resolution order: `process.env` > shared varlock mesh config (`.env.local` over `.env.schema`) > legacy `~/.codex/mcp-gateway.env`
- Project root: `/Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway`

## Commands
```bash
# Store a secret
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs set KEY value

# Read a secret
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs get KEY

# Delete a secret
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs del KEY

# List all stored secrets
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs list

# Show source/config status for managed keys
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs status

# Inspect Claude plugin OAuth state for supported imports
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs claude-status

# Scan all approved secret/config sources before importing
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs scan-sources

# Import supported secrets from codex auth + legacy env only
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs import-codex

# Import approved secrets from local sources and refresh the config manifest
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs import-sources

# Refresh the non-secret operational config manifest
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs import-config

# Import live supported Claude plugin tokens into mesh .env.local
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs import-claude

# Migrate legacy ~/.codex/mcp-gateway.env values into mesh .env.local
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs migrate-legacy

# Sync mesh config to peer nodes
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs sync-mesh

# Show resolved environment (secrets masked)
node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs dump
```

## Key Discovery Procedure
When asked to find and store keys, scan these sources in order:

1. **ShadowVault** (Google Sheet): Open in Chrome, read the Credentials tab
2. **Shared mesh config**: inspect `/Users/Shared/shadow-labs/mesh-config/.env.local` and `.env.schema`
3. **Local .env files**: `find /Users/sdluffy -maxdepth 4 -name ".env" | grep -v node_modules`
4. **JARVIS .env files**: `ssh jarvis-local 'find ~ -maxdepth 4 -name ".env" | grep -v node_modules'`
5. **ULTRON .env files**: `ssh x-1-lan 'find ~ -maxdepth 4 -name ".env" | grep -v node_modules'`
6. **Environment variables**: `env | grep -iE "(api|key|token|secret)"`
7. **Claude plugin OAuth metadata**: inspect `~/.claude/.credentials.json` via `claude-status` or `import-claude`

## Known Key Names
- `SMARTSHEET_API_TOKEN` — Smartsheet API access
- `NOTION_API_TOKEN` — Notion integration
- `DROPBOX_ACCESS_TOKEN` — Dropbox API access for gateway and agent exchange
- `NOTION_API_TOKEN_WORK` — Notion work profile
- `NOTION_API_TOKEN_PERSONAL` — Notion personal profile
- `MCP_GATEWAY_API_KEY` — Gateway auth
- `OPENAI_API_KEY` — OpenAI/GPT access
- `OPENROUTER_API_KEY` — OpenRouter (multi-model)
- `DEEPSEEK_API_KEY` — DeepSeek
- `GROQ_API_KEY` — Groq fast inference
- `GOOGLE_AI_API_KEY` — Google Gemini
- `ANTHROPIC_API_KEY` — Anthropic (usually via OAuth, not stored)
- `HF_TOKEN` — HuggingFace
- `BRAVE_API_KEY` — Brave search
- `PINECONE_API_KEY` — Pinecone plugin/provider access
- `GREPTILE_API_KEY` — Greptile plugin/provider access
- `GLM_API_KEY` / `ZAI_API_KEY` — GLM / z.ai direct access
- `MINIO_ACCESS_KEY` / `MINIO_SECRET_KEY` — project-local object storage credentials
- `SIGNING_SECRET` — project-local signing secret

## Ollama / ai-fi gateway (Mac Mini)
- Ralph Loop example uses **Ollama on FRIDAY** (Mac Mini): `~/.cursor/examples/ralph-loop-basic.mts`.
- Default base: `http://friday-mac.local:11434` (script appends `/api`). Override with `OLLAMA_BASE_URL` or `AIFI_OLLAMA_BASE_URL`.
- Model: `OLLAMA_MODEL` (default `qwen3:8b`). On the Mac Mini run `ollama pull <model>` so the model is available.
- No API key needed for this path; keys in mesh config apply to MCP gateway (e.g. `MCP_GATEWAY_API_KEY`), not Ollama.

## After Storing Keys
1. Run `node /Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs dump` to verify
2. Sync mesh config to peers when needed with `/Users/Shared/shadow-labs/mesh-config/varlock-mesh-sync`
3. Keys become available through `loadEnv()` in `mcp-gateway` bootstrap before services initialize

## Claude Import Notes
- `claude-status` reads Claude plugin OAuth metadata safely and reports whether any supported connector is importable
- `import-claude` never prints token values; it only writes supported live access tokens into mesh `.env.local`
- Current supported mapping is `plugin:Notion:notion -> NOTION_API_TOKEN`
- If Claude only has discovery metadata or expired/no-token entries, nothing will be imported

## Pipeline

```
Input ("set key" / "get key" / "scan" / "sync" / “find key” / secret management task)
  ↓
Resolution (process.env → mesh .env.local → mesh .env.schema → legacy ~/.codex/mcp-gateway.env)
  ↓
Operation (get / set / del / list / scan-sources / import / sync-mesh)
  ↓
Verification (dump to confirm, status check)
  ↓
Artifact Routing
  ├── /Users/Shared/shadow-labs/mesh-config/.env.local  (primary secret store)
  ├── /Users/Shared/shadow-labs/mesh-config/.env.schema  (schema/defaults)
  └── Peer nodes via varlock-mesh-sync  (cross-mesh propagation)
```

## Modes

### Default: Get/Set
- Single key operation: `ai-fi get KEY` or `ai-fi set KEY value`.

### scan
- `ai-fi scan-sources`: Discover keys from all approved sources without importing.

### import
- `ai-fi import-sources`: Import approved secrets from local sources into mesh config.

### status
- `ai-fi status`: Show which keys are resolved, from which source, and any gaps.

### dump
- `ai-fi dump`: Show full resolved environment with secrets masked. For verification only.

### sync
- `ai-fi sync-mesh`: Push mesh config to peer nodes.

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Primary secrets | `/Users/Shared/shadow-labs/mesh-config/.env.local` | Live secret store |
| Schema/defaults | `/Users/Shared/shadow-labs/mesh-config/.env.schema` | Key schema and defaults |
| Config manifest | `ai-fi.config-manifest.json` in mesh config root | Non-secret operational config |
| Legacy fallback | `~/.codex/mcp-gateway.env` | Pre-migration secrets (deprecated) |

Secrets are never written to `/tmp`, chat output, or ShadowArchive reports.

## Fallback Chain

1. **Primary: Mesh varlock config** — `/Users/Shared/shadow-labs/mesh-config/.env.local`
2. **Mesh schema** — `.env.schema` for defaults when `.env.local` doesn't have a key
3. **Legacy env** — `~/.codex/mcp-gateway.env` for pre-migration keys
4. **Live discovery** — `scan-sources` to find keys in environment, Claude metadata, or other `.env` files
5. **Operator prompt** — When a key cannot be found anywhere, ask the operator to provide it

## Prerequisites

- `ai-fi.mjs` module at `/Users/sdluffy/Spaces/Personal/Projects/_mcp-and-agents/mcp-gateway/src/ai-fi.mjs`
- Mesh config directory writable: `/Users/Shared/shadow-labs/mesh-config/`
- Node.js runtime for `ai-fi.mjs`
- `gcloud` CLI (optional, for Google Sheets-based ShadowVault)
- SSH access to peer nodes (for `sync-mesh`)

## Error Handling

| Failure | Recovery |
|---------|----------|
| Mesh config directory not writable | Fall back to legacy `~/.codex/mcp-gateway.env`; note degraded mode |
| Key not found in any source | Report missing key; suggest `scan-sources` or operator prompt |
| `import-claude` finds no live tokens | Report that Claude has no importable tokens; suggest manual OAuth refresh |
| `sync-mesh` peer unreachable | Log unreachable peer; sync to available peers; report partial sync |
| `claude-status` metadata unreadable | Report corrupted metadata; suggest re-auth via Claude UI |
| Duplicate key in multiple sources | Prefer `.env.local` over `.env.schema` over legacy; report resolution path |

## Contract

- **Secrets never in chat output:** Never print API key values in chat. Use `dump` (masked) for verification only.
- **Secrets never in ShadowArchive:** Do not write secrets to reports, training pairs, or any persistent log.
- **Resolution order is strict:** `process.env` > `.env.local` > `.env.schema` > legacy. Do not skip tiers.
- **Import is opt-in:** `scan-sources` discovers but does not import. `import-sources` imports approved keys only.
- **Claude import never prints tokens:** `import-claude` writes to `.env.local` without printing token values.
- **Mesh sync requires connectivity:** `sync-mesh` does not silently skip peers. It reports partial sync.
- **No secrets in routing logs:** The `~/.shadow-connect/routing.jsonl` must not contain API key values.
- **SHADOW separation:** Keep mesh topology and peer node addresses in SHADOW-local config only.

## Related Skills

- `acp-debug` — fix provider failures
- `multi-model-query` — fan-out to multiple providers simultaneously
- `shadow-connect` — zone-aware routing layer
- `aacp` — access control for sensitive resource delegation
- `mesh-ops` — check node health before routing to remote providers
