---
name: SHADOW Service Discovery
description: 'Use when the user needs: Validate LaunchAgent plists match actual binaries'
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-service-discovery
  icon_style: craft-category-glyph-v1
---

# shadow-service-discovery

Service discovery validation — check declared services vs installed binaries.

## When to Use

Use when:
- Services fail to start due to missing binaries
- Validating LaunchAgent configuration
- Detecting drift between declared services and installed packages
- After system updates or package installations

## Pattern

Automated plist validation with binary existence checking:

1. Extract binary path from plist (Program or ProgramArguments)
2. Validate file existence
3. Check executable permission
4. Report mismatches

## Usage

```bash
shadow-service-discovery-validate
```

## Output

ANSI-formatted report with:
- Valid: Binary exists and executable
- Missing: Binary not found
- Mismatched: Binary exists but wrong permissions

Counts: X valid, Y missing, Z mismatched

## Failure Categories

- **Missing binary**: Plist points to non-existent file
- **Not executable**: File exists but lacks execute permission
- **Wrong path**: Binary moved or package updated

## Remediation

- Missing: Install package via Homebrew/apt
- Not executable: `chmod +x` the binary
- Wrong path: Update plist with correct path

## Integration

Located at: `scripts/service-discovery-validate.sh`

Uses `ansi-status-lib.sh` for consistent visual language.

Part of "Service Discovery via Absence" pattern from Shadow Workflow Catalog.
