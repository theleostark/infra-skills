---
name: SHADOW Self Healing Services
description: 'Use when the user needs: Auto-fix common LaunchAgent service issues'
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-self-healing-services
  icon_style: craft-category-glyph-v1
---

# shadow-self-healing-services

Self-healing service discovery — detect and auto-fix common LaunchAgent issues.

## When to Use

Use when:
- Services fail to start due to missing binaries
- Binaries exist but lack execute permissions
- Services need restart after fixes
- Proactive maintenance to prevent service failures

## Pattern

Detect common issues and auto-remediate:

1. **Missing binary** → Auto-install via brew/npm
2. **Not executable** → Auto-fix permissions (chmod +x)
3. **Service down** → Auto-restart after fix
4. **Missing plist** → Manual intervention required

## Usage

```bash
shadow-service-autofix           # Live mode - makes changes
shadow-service-autofix --dry-run  # Preview what would be fixed
```

## Auto-Fix Capabilities

**Can fix:**
- Missing binaries (auto-install via Homebrew/npm)
- Permission issues (auto-chmod +x)
- Service restart (launchctl unload/load)

**Cannot fix:**
- Missing plist files (manual recreation required)
- No Program/ProgramArguments key (manual configuration)
- System services (com.apple.*)

## Output

ANSI-formatted report with:
- ✓ Fixed issues (green)
- ✗ Failed issues (red)
- Summary of agents processed
- Count of valid/fixed/failed services

## Safety

- Dry-run mode for preview
- Changes require user confirmation (remove --dry-run)
- Conservative approach: only fixes obvious issues
- Preserves manual configuration for complex cases

## Integration

Located at: `scripts/service-discovery-autofix.sh`

Uses `ansi-status-lib.sh` for consistent visual language.
Extends `shadow-service-discovery` skill with auto-fix capability.

Part of self-healing patterns from Shadow operational automation.
