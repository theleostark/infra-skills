---
name: SHADOW Self-Improve
description: "Use when proposing automated improvements to SHADOW itself — test additions, schema tightenings, .gitignore expansions, CI hardening based on observed failure modes. Agent-initiated maintenance. Trigger on self-improve, auto-fix, propose test, observed failure, maintenance, harden CI."
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-self-improve
  icon_style: craft-category-glyph-v1
  source_pattern: "20: Self-Improving Loop"
---

# shadow-self-improve — Agent-Initiated Self-Improvement

## When to Use

Invoke this skill when:
- A CI failure reveals a gap in test coverage
- A symlink break could have been caught earlier
- A registry schema allows invalid entries through
- An `.gitignore` whitelist is too permissive or missing entries
- A skill trigger fires incorrectly or misses a valid case
- Operator says "self-improve", "auto-fix", "propose test", "harden CI", "maintenance"

## Pattern

The highest-leverage project is recursively removing humans from the improvement cycle. Each generation of model/system assists in producing the next. The flywheel accelerates when the human bottleneck is eliminated from eval, data curation, and alignment.

Applied to SHADOW: the system observes its own failure modes and proposes fixes, which a human approves. Over time, the approval rate should increase as proposals get better.

## Loop

```
1. OBSERVE — run verify:portable, CI, redteam probe
2. CLASSIFY — categorize each failure by pattern class
3. PROPOSE — generate specific fix (test addition, schema tightening, config patch)
4. SUBMIT — create PR or present proposal to operator
5. APPROVE — operator approves or rejects with feedback
6. APPLY — execute approved changes
7. RE-VERIFY — confirm fix works without regression
8. LEARN — encode approval/rejection pattern for future proposals
```

## Proposal Types

### Test Addition

When a failure mode reveals missing coverage:

```javascript
// Proposed: test/symlink-graph.test.js
// Pattern class: broken symlink target
// Triggered by: verify:portable failure on docs/reference/BRAND.md

test('all repo symlinks have valid targets', () => {
  const symlinks = findSymlinks('.');
  for (const link of symlinks) {
    expect(fs.existsSync(fs.readlinkSync(link))).toBe(true);
  }
});
```

### Schema Tightening

When registry allows invalid entries:

```json
// Proposed: tighten program-registry.json schema
// Pattern class: missing required field
// Triggered by: program with model-code but no dropbox_path

{
  "required": ["program_id", "model_code", "dropbox_path", "status"],
  "additionalProperties": false
}
```

### .gitignore Expansion

When an untracked path should be whitelisted or blocked:

```gitignore
# Proposed: whitelist new tracked path
# Pattern class: new skill directory not in gitignore
# Triggered by: `git status` showing untracked skills/ path

!skills/shadow-redteam/
```

### CI Hardening

When CI could catch more failure classes:

```yaml
# Proposed: add registry validate to pretest
# Pattern class: invalid JSON in registry
# Triggered by: program-registry.json parse error

"pretest": "node tools/program-registry/index.mjs validate && npm run verify:portable"
```

## Proposal Format

```markdown
## 🔧 Self-Improve Proposal

**Observed failure:** verify:portable reported 3 broken symlinks
**Pattern class:** symlink target drift after directory rename
**Frequency:** 3rd occurrence this month

### Proposed Fix
- Add `test/symlink-targets.test.js` to pretest
- Retarget broken symlinks: `BRAND.md`, `SKILLS.md`, `WORKFLOW.md`

### Expected Impact
- Catches symlink break on `npm test` before push
- Prevents CI failure on portable-surface-check

### Risk
- Minimal — additive test, no behavior change
- 5-line test file, no dependencies
```

## Non-Negotiables

- **Propose, never auto-apply** — all changes require operator approval
- **Minimal proposals** — smallest fix that addresses the pattern class
- **Re-verify after apply** — always run `verify:portable` + `npm test`
- **Track approval rate** — if proposals keep getting rejected, adjust proposal quality
- **No breaking changes** — proposals must not break existing functionality

## Integration

- `shadow-redteam` — feeds findings into self-improve proposals
- `verify:portable` — baseline verification before and after
- `shadow-patterns` — pattern library informs classification of failure modes
- `crystallize` — if a proposal class recurs, crystallize it as a new pattern

## Crystallized From

- Pattern 20: Self-Improving Loop (SHADOW Patterns Library)
- Source: YouTube analysis 2026-05-03, "Nat Friedman and Daniel Gross with Collisons"
- Insight: removing humans from the improvement cycle is the prime project
