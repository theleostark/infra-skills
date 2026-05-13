---
name: SHADOW Smt Health
description: 'Use when the user needs: Multi-layer health triangulation for mesh services'
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-smt-health
  icon_style: craft-category-glyph-v1
---

# shadow-smt-health

Multi-layer health triangulation for mesh services and APIs.

## When to Use

Use when a service health check fails and you need to diagnose from multiple angles.

## Pattern

Multi-layer testing with decision tree analysis:

1. **Layer 1: External test** - curl from local machine
2. **Layer 2: Internal test** - SSH → curl localhost
3. **Layer 3: Registry check** - Tailscale status
4. **Layer 4: Process check** - lsof for port binding

## Usage

```bash
shadow-health-triangulate "API" "100.76.188.80" "8787" "leo"
```

## Parameters

- **Service name**: Human-readable name (e.g., "API", "Database")
- **Host**: IP address or hostname
- **Port**: Service port number
- **SSH user**: Remote user for internal testing

## Output

ANSI-formatted diagnosis with:
- ✓/✗ status for each layer
- Root cause identification
- Remediation steps

## Failure Categories

- **Binding issue**: Port not listening, process not running
- **Registry mismatch**: Node not in Tailscale mesh
- **Service down**: Process exists but not responding
- **Health check failure**: Service up but health endpoint fails

## Integration

Located at: `scripts/health-triangulate.sh`

Uses `ansi-status-lib.sh` for consistent visual language.
