---
name: SHADOW Research Factory
description: 'Use when the user needs: The closed-loop system that connects shadow-connect (routing) → shadow-research (training) → model-registry (deployment) → shadow-connect (improved routing). The factory IS the product. Use when checking factory status, triggering training runs, reviewing DOE results, syncing models to mesh, or debugging the pipeline. Trigger on "factory", "training pipeline", "DOE results", "model sync", "research loop", "ralph-loop", "taxonomy", "training data".'
icon: icon.svg
allowed-tools: Bash(*), Read, Write
metadata:
  shadow_family: infrastructure
  shadow_role: orchestrator
  shadow_scope: system
  capability_summary: Closed-loop SLM training + deployment factory
  depends_on:
  - shadow-connect
  - shadow-research (spec)
  - model-registry.yaml
  openclaw_surfaces:
  - codex
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-research-factory
  icon_style: craft-category-glyph-v1
---

# shadow-research-factory

The closed-loop that makes Shadow Lab self-improving. Every agent session generates training data that improves local models that make future sessions better.

## The Loop

```
shadow-connect (User Layer)
    │ routing decisions → routing.jsonl
    │ zone detections → taxonomy.jsonl
    │ skill dispatches → routing.jsonl
    ▼
shadow-research Phase 1 (Crawl + Distill)
    │ kernel-diff.jsonl → classified entries
    │ training-pairs.jsonl → Q/A extraction
    ▼
shadow-research Phase 2 (DOE Experiment)
    │ 2^4 factorial matrix → optimal config
    │ PI = accuracy + coverage + latency + cost
    ▼
shadow-research Phase 3 (Train + Deploy)
    │ Unsloth RL on AURION → new SLM weights
    │ Quantize per device (INT8/Q4/FP16)
    │ rsync to mesh nodes
    ▼
model-registry.yaml (Compute Layer)
    │ status: deployed
    │ MQTT: shadow/model/updated
    ▼
shadow-connect (improved routing)
    └─── loop restarts ───┘

tool orchestration mining (NEW)
    │ session memories → repeated patterns
    │ pattern-candidates.jsonl → skill drafts
    │ MCP: research.mine_patterns
    ▼
skill auto-generation
    │ uncovered patterns → SKILL.md drafts
    │ in /Volumes/SHADOW/shadow-lab/skills/
    └─── factory teaches itself new skills ───┘
```

## Factory Status Check

To check the factory health:

```bash
# 1. Queue depth — how many unprocessed kernel diffs?
wc -l ~/.shadow-research/queue/kernel-diff.jsonl 2>/dev/null || echo "0 entries"

# 2. Last anchor — when was the last kernel capture?
python3 -c "
import json
from pathlib import Path
state = Path.home() / '.shadow-research' / 'anchor-state.json'
if state.exists():
    d = json.loads(state.read_text())
    print(f'Last anchor: {d.get(\"timestamp\", \"unknown\")}')
    print(f'Kernel hash: {d.get(\"kernel_hash\", \"unknown\")[:16]}...')
else:
    print('No anchor state — hooks may not be firing')
"

# 3. Model registry status
python3 -c "
import yaml
from pathlib import Path
reg = Path('/Volumes/SHADOW/ShadowArchive/10-projects/ANE-LM/model-registry.yaml')
if reg.exists():
    d = yaml.safe_load(reg.read_text())
    for name, spec in d.get('models', {}).items():
        print(f'{name}: {spec.get(\"status\", \"unknown\")} ({spec.get(\"params\", \"?\")})')
else:
    print('No model registry')
"

# 4. Fleet status
tailscale status 2>/dev/null | grep -E "jarvis|friday|aurion|ultron" || echo "Tailscale unavailable"

# 5. Training data inventory
echo "=== Training Data ==="
for f in /Volumes/SHADOW/ShadowArchive/10-projects/ANE-LM/training/*.jsonl; do
  [ -f "$f" ] && echo "$(basename $f): $(wc -l < "$f") entries"
done 2>/dev/null || echo "No training data yet"
```

## Training Data Sources

### From shadow-connect (automatic)

| Source | Output | Trains |
|--------|--------|--------|
| Zone detection | taxonomy.jsonl | slm-pico (classifier) |
| Routing decisions | routing.jsonl | slm-nano (decomposer) |
| Skill dispatches | routing.jsonl | slm-nano |
| Session Q/A | instructions.jsonl | slm-micro (synthesizer) |
| Code patterns | code_patterns.jsonl | slm-micro |
| Task estimates | estimates.jsonl | slm-nano (calibration) |

### From shadow-research hooks (automatic)

| Hook | Captures | Output |
|------|----------|--------|
| PreCompact | Kernel hash, training pairs, context summary | kernel-diff.jsonl |
| PostCompact | Delta (what changed during compaction) | kernel-diff.jsonl |
| Stop | Session summary, decisions, concepts | kernel-diff.jsonl |

### From hl-fetch / shadow-feed (on demand)

| Source | When | Output |
|--------|------|--------|
| hl-fetch | URL extraction flagged as kernel-worthy | kernel-diff.jsonl |
| shadow-feed | Lane-classified content | daily digests → training pairs |

## Generate Training Data

### taxonomy.jsonl (for slm-pico zone classifier)

```bash
python3 /Volumes/SHADOW/ShadowArchive/10-projects/ANE-LM/training/gen-taxonomy.py
```

This script crawls:
- `~/.shadow-connect/profiles.yaml` → zone keyword definitions
- `~/.Codex/skills/*/SKILL.md` → skill metadata (shadow_family, names)
- `~/.Codex/projects/-Volumes-SHADOW/memory/*.md` → memory file names → domain terms
- `/Volumes/SHADOW/ShadowArchive/10-projects/shadow-feed/ingest.sh` → lane keywords

Output: `training/taxonomy.jsonl` with entries like:
```jsonl
{"term": "FH4S", "domain": "enterprise", "parent": "astemo", "source": "profiles.yaml"}
{"term": "shadow-top", "domain": "developer", "parent": "infrastructure", "source": "skill:shadow-top"}
{"term": "YouTube", "domain": "personal", "parent": "social", "source": "skill:hl-youtube"}
```

### routing.jsonl (from shadow-connect decisions)

Generated automatically by shadow-connect during sessions. Each routing decision appends:
```jsonl
{"query_hash": "abc123", "zone": "developer", "skill": "shadow-engg", "model": "cloud", "latency_ms": 450, "ts": "2026-03-25T18:00:00Z"}
```

### instructions.jsonl (from session Q/A pairs)

Generated by shadow-research PreCompact hook. Each compaction captures:
```jsonl
{"q": "How do I wire INT8 kernels for ANE?", "a": "Use CPU dequant fallback — private ANE compiler lacks constexpr_affine_dequantize...", "domain": "developer"}
```

## Trigger Training Run (on AURION)

```bash
# SSH to AURION and start training
ssh aurion "cd /data/shadow-research && python3 train-pico.py \
  --data /data/shadow-research/training/taxonomy.jsonl \
  --base TinyLlama-1.1B \
  --epochs 3 \
  --budget 300 \
  --output /data/models/slm-pico/v1/"
```

## Deploy New Model

After training completes on AURION:

```bash
# 1. Quantize for JARVIS (INT8 CoreML)
ssh aurion "python3 /data/shadow-research/quantize.py \
  --model /data/models/slm-pico/v1/ \
  --format coreml --quant INT8 \
  --output /data/models/slm-pico/v1-int8/"

# 2. Quantize for FRIDAY (GGUF Q4)
ssh aurion "python3 /data/shadow-research/quantize.py \
  --model /data/models/slm-pico/v1/ \
  --format gguf --quant Q4_K_M \
  --output /data/models/slm-pico/v1-q4/"

# 3. Sync to JARVIS
rsync -avz aurion:/data/models/slm-pico/v1-int8/ \
  /Volumes/SHADOW/ShadowArchive/10-projects/ANE-LM/models/slm-pico/

# 4. Sync to FRIDAY
ssh aurion "rsync -avz /data/models/slm-pico/v1-q4/ friday:/data/models/slm-pico/"

# 5. Update registry
python3 -c "
import yaml
reg_path = '/Volumes/SHADOW/ShadowArchive/10-projects/ANE-LM/model-registry.yaml'
with open(reg_path) as f:
    reg = yaml.safe_load(f)
reg['models']['slm-pico']['status'] = 'deployed'
with open(reg_path, 'w') as f:
    yaml.dump(reg, f, default_flow_style=False)
print('Registry updated: slm-pico → deployed')
"
```

## DOE Matrix Status

Check current DOE experiment progress:

```bash
cat ~/.shadow-research/queue/doe-results.jsonl 2>/dev/null | python3 -c "
import sys, json
results = [json.loads(l) for l in sys.stdin if l.strip()]
if not results:
    print('No DOE results yet — Round 0 not started')
else:
    print(f'{len(results)} runs completed')
    best = max(results, key=lambda r: r.get('pi', 0))
    print(f'Best PI: {best[\"pi\"]:.3f} (config: {best.get(\"config\", \"unknown\")})')
" 2>/dev/null
```

## Factory Metrics

| Metric | Target | How to Measure |
|--------|--------|---------------|
| Kernel freshness | <10 min lag | Compare anchor timestamp to now |
| Queue depth | <50 entries | `wc -l kernel-diff.jsonl` |
| Training frequency | Weekly | Check last model deploy timestamp |
| Zone accuracy | >85% | slm-pico eval on 100 test queries |
| Routing accuracy | >80% | slm-nano eval on routing decisions |
| Quality gap | <0.15 | Local vs cloud answer comparison |

## Pipeline

```
Input (trigger: factory, training pipeline, DOE results, model sync)
  ↓
Queue Check (kernel-diff.jsonl depth, last anchor, pending signals)
  ↓
Route to Phase:
  ├── Phase 1: Crawl + Distill (kernel-diff → classified → training-pairs)
  ├── Phase 2: DOE Experiment (2^4 factorial → optimal config)
  └── Phase 3: Train + Deploy (Unsloth RL → quantize → rsync to mesh)
  ↓
Model Registry Update (model-registry.yaml → status: deployed)
  ↓
Loop Closes (shadow-connect uses improved routing → generates more data)
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Full factory status: queue depth, last anchor, model registry, fleet, training data | `factory`, `training pipeline`, `pipeline status` |
| `status` | Quick health check — queue depth + last anchor timestamp | `factory status` |
| `queue` | Kernel-diff queue depth and oldest entry | `kernel-diff`, `research queue` |
| `generate` | Generate training data (taxonomy, routing, instructions) | `generate training data`, `taxonomy` |
| `train` | Trigger training run on AURION | `trigger training`, `train model` |
| `deploy` | Deploy trained model to mesh nodes | `deploy model`, `model sync` |
| `doe` | DOE experiment status and results | `DOE results`, `experiment status` |
| `sync` | Sync training data to AURION | `sync to aurion`, `ralph-loop` |

## Artifact Routing

| Artifact | Path | Purpose |
|----------|------|----------|
| Taxonomy training data | `training/taxonomy.jsonl` | Zone classifier training |
| Routing training data | `training/routing.jsonl` | Decomposer training |
| Instruction pairs | `training/instructions.jsonl` | Synthesizer training |
| Kernel diff queue | `~/.shadow-research/queue/kernel-diff.jsonl` | Session capture feed |
| Model registry | `ANE-LM/model-registry.yaml` | Model deployment state |
| DOE results | `~/.shadow-research/queue/doe-results.jsonl` | Experiment outcomes |

## Fallback Chain

1. **Primary:** Local queue → sync.sh → AURION training → quantize → deploy → update registry
2. **AURION unreachable:** Queue locally; report AURION offline; suggest `ssh aurion` to check
3. **Sync script fails:** Manual `rsync` of training data; note degraded sync
4. **Training fails on AURION:** Check GPU availability; check data quality; report failure with logs
5. **Quantization fails:** Deploy unquantized model; note larger footprint
6. **Mesh node unreachable for deploy:** Skip that node; deploy to reachable nodes; note pending
7. **Last resort:** Queue only; report that training/deploy pipeline is blocked

## Prerequisites

- `~/.shadow-research/queue/` writable for kernel-diff.jsonl
- AURION accessible via SSH for training runs
- Tailscale mesh active for node deployment
- `bun` runtime for shadow-mcp (if using MCP tools)
- Python with `yaml`, `httpx` for queue inspection and training scripts
- Ollama on FRIDAY for free fallback training (when GLM balance empty)
- GLM/Zhipu API key in keychain (optional, balance currently empty)

## Error Handling

| Failure | Recovery |
|---------|----------|
| AURION SSH timeout | Report; queue locally; suggest checking AURION docker/services |
| kernel-diff.jsonl corrupt | Parse what's valid; quarantine corrupt entries; report line numbers |
| Model registry YAML parse error | Report; do not overwrite; suggest manual fix |
| Training run OOM on AURION | Reduce batch size; use smaller base model; report with config |
| rsync to FRIDAY fails | Check Tailscale status; retry once; skip FRIDAY; note pending |
| No training data generated | Check hooks (shadow-anchor); check that sessions are producing captures |
| GLM balance empty | Use Ollama qwen3:8b on FRIDAY as free fallback; note degraded quality |

## Contract

- **The factory IS the product.** This closed-loop system (routing → training → deployment → improved routing) is the core value proposition.
- **No public dataset publishing.** Training data stays internal. Use `opentraces` for any public sharing.
- **No credential exposure.** Factory reports must not contain API keys, SSH keys, or GLM tokens.
- **Queue monitoring is continuous.** If queue depth exceeds 50, flag in output.
- **Model registry is source of truth.** Always read registry before assuming a model's status.
- **Externalization rule.** DOE results and factory status go to `ShadowArchive/80-reports/` when generating reports.
- **Do not** deploy untested models to production mesh nodes.
- **Do not** auto-trigger training without operator awareness (training consumes AURION GPU).
- **Do not** overwrite model registry without backup of previous state.
