---
name: SHADOW Report Externalization
description: Use when generating structured reports from operational data
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-report-externalization
  icon_style: craft-category-glyph-v1
---

# shadow-report-externalization

Template-based report generation — turn operational data into documented markdown.

## When to Use

Use when:
- Completing operational tasks and need documentation trail
- Externalizing system state for audit/review
- Creating weekly/monthly health reports
- Handoff documentation between shifts or teams

## Pattern

Template-based report generation with live data capture:

1. Capture current system state
2. Apply structured template
3. Calculate health scores/metrics
4. Generate action items based on status
5. Route to destination (Obsidian, file, stdout)

## Usage

```bash
shadow-report smt-health [output-file.md]
shadow-report service-audit [output-file.md]
shadow-report bookmark-phase [output-file.md]
```

## Report Types

**smt-health**: Mesh nodes, API status, disk usage, recommendations
- Health score: 4 metrics × 25%
- Action items: API down, disk pressure, OpenAI proxy

**service-audit**: LaunchAgent validation findings
- Valid/missing/mismatched counts
- Remediation recommendations

**bookmark-phase**: Phase analysis with action items
- Topic distribution analysis
- Workflow phase indicators

## Output

Markdown-formatted reports with:
- Timestamp and metadata
- Executive summary
- Detailed status sections
- Recommendations with priority

## Integration

Located at: `scripts/report-template.sh`

Uses `ansi-status-lib.sh` for consistent visual language.

Externalization as Process Completion pattern from Shadow Workflow Catalog.
