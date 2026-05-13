---
name: SHADOW Self Healing Systemd
description: 'Use when the user needs: Auto-fix common systemd service issues'
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-self-healing-systemd
  icon_style: craft-category-glyph-v1
---

# shadow-self-healing-systemd

Self-healing systemd service management — detect and auto-fix common service issues.

## When to Use

Use when:
- Services fail to start due to missing binaries
- Binaries exist but lack execute permissions
- Services need restart after fixes
- Proactive maintenance to prevent service failures

## Pattern

Detect common issues and auto-remediate:

1. **Missing binary** → Auto-install via apt/yum/npm
2. **Not executable** → Auto-fix permissions (chmod +x)
3. **Service down** → Auto-restart after fix
4. **Missing ExecStart** → Manual intervention required

## Usage

```bash
shadow-systemd-autofix              # Live mode - makes changes
shadow-systemd-autofix --dry-run     # Preview what would be fixed
shadow-systemd-autofix openclaw-*    # Target specific services
```

## Auto-Fix Capabilities

**Can fix:**
- Missing binaries (auto-install via apt/yum/npm)
- Permission issues (auto-chmod +x)
- Service restart (systemctl restart)

**Cannot fix:**
- Missing ExecStart directive (manual configuration required)
- System services (requires root)
- Service file syntax errors (manual edit required)

## Output

ANSI-formatted report with:
- ✓ Fixed issues (green)
- ✗ Failed issues (red)
- Summary of services processed
- Count of valid/fixed/failed services

## Safety

- Dry-run mode for preview
- Changes require root privileges (sudo)
- Conservative approach: only fixes obvious issues
- Preserves manual configuration for complex cases

## Integration

Located at: `scripts/systemd-service-autofix.sh`

Uses `ansi-status-lib.sh` for consistent visual language.
Extends `shadow-self-healing-services` skill to systemd architecture.
Part of self-healing patterns from Shadow operational automation.

## Architecture Differences

Compared to LaunchAgent self-healing:

| LaunchAgent | Systemd |
|-------------|---------|
| launchctl load/unload | systemctl restart |
| ~/Library/LaunchAgents/ | /etc/systemd/system/, /usr/lib/systemd/system/ |
| Program key | ExecStart directive |
| com.apple.* exclusion | No system service exclusion |
