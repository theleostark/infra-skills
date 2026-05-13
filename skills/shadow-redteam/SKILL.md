---
name: SHADOW Redteam
description: "Use when continuously adversarially hardening SHADOW surfaces — fuzz registries, probe symlinks, break skill triggers, test A2A queues. Runs verify → adversarially probe → fix in background. Trigger on redteam, adversarial hardening, continuous security, fuzz, probe, security test."
icon: icon.svg
metadata:
  category: mesh/shadow
  family: shadow
  lifecycle: active
  canonical_slug: shadow-redteam
  icon_style: craft-category-glyph-v1
  source_pattern: "22: Continuous Adversarial Hardening"
---

# shadow-redteam — Continuous Adversarial Hardening

## When to Use

Invoke this skill when:
- Running security verification on SHADOW surfaces
- Fuzzing program registries, symlink graphs, or skill triggers
- Testing A2A handoff queue integrity
- Probing for stale configs or broken references
- Hardening CI pipelines against edge cases
- Operator says "redteam", "adversarial", "fuzz", "probe", "security test"

## Pattern

Security shifts from periodic pen-test events to continuous AI-driven red-teaming. Asymmetry favors the defender: the system attacks itself at machine speed, finds weaknesses, and patches before external actors exploit.

The verify/repair loop upgrades from two-state to three-state:

```
verify → adversarially probe → fix
```

## Probe Surfaces

| Surface | What to Probe | Failure Mode |
|---------|---------------|-------------|
| Symlink graph | Broken targets, circular refs, missing volumes | `verify:portable` fails |
| Program registry | Invalid schema, missing `dropbox_path`, orphan programs | JSON parse error |
| Skill triggers | Orphan SKILL.md, stale `skills-lock.json`, missing deps | Skill not found |
| A2A queues | Stale handoff files, orphan tasks, stuck locks | Queue not draining |
| CI pipeline | Missing test coverage, skipped asserts, flaky gates | `npm test` passes but shouldn't |
| Config trees | Callback endpoints, webhook paths, bind settings | `hl-shadowprivacy` surface |
| Artifact routing | Missing Dropbox roots, broken relay paths | File saved nowhere |

## Execution Modes

### Quick Scan (default)

```bash
# Run all probe surfaces, report findings
npm run verify:portable                    # symlink + registry baseline
bash scripts/verify-root-doc-symlinks.sh   # doc symlink integrity
```

Then probe deeper:

1. **Registry fuzz**: iterate every entry in `program-registry.json`, validate schema, check `dropbox_path` exists, verify model-code format
2. **Symlink walk**: find all symlinks under repo root, check targets exist and aren't circular
3. **Skill audit**: list all SKILL.md files, check `skills-lock.json` consistency, verify icon files exist
4. **Config probe**: scan `.shadow/` configs for stale references, orphan endpoints

### Deep Probe

For each finding from Quick Scan, attempt exploitation:

- Can a broken symlink chain corrupt `npm test`?
- Can a stale `skills-lock.json` entry cause wrong skill to fire?
- Can an orphan program-registry entry route artifacts to nowhere?
- Can a missing `.gitignore` whitelist leak sensitive paths?

### Auto-Fix

When a probe finds a fixable issue:

1. **Report** — state the finding, severity, affected surface
2. **Propose** — generate the specific fix (edit command, missing file, config patch)
3. **Apply** (with operator approval) — execute the fix
4. **Re-verify** — run `verify:portable` to confirm fix didn't break anything
5. **Add test** — propose a regression test that would catch this class of failure

## Output Format

```markdown
## 🔴 SHADOW Redteam Report

| # | Surface | Finding | Severity | Status |
|---|---------|---------|----------|--------|
| 1 | Symlink graph | 3 broken targets under `docs/reference/` | Medium | Fixed + test proposed |
| 2 | Program registry | `4G0A` missing `dropbox_path` | High | Reported, needs operator input |
| 3 | Skill lock | 2 stale entries in skills-lock.json | Low | Auto-fixed |

### Fixes Applied
- `docs/reference/BRAND.md` → retargeted to `reference/BRAND.md`
- `skills-lock.json` entries for `_archived/` skills removed

### Tests Proposed
- `test/symlink-graph.test.js` — walk all symlinks, assert targets exist
```

## Non-Negotiables

- **Never auto-delete** — only auto-fix trivially safe issues (symlinks, lock entries). Escalate everything else.
- **Always re-verify** — after any fix, run full `verify:portable` cycle.
- **Report before fix** — operator sees findings before any changes are applied.
- **No external attack** — this is self-probing only. No network scanning, no external penetration testing.

## Integration with Existing Skills

- `hl-shadowprivacy` — feeds privacy surface findings into redteam report
- `hl-shadowsecure` — feeds exposure findings into redteam report
- `shadow-os-skills` — symlink graph probe uses OS-level walk
- `verify:portable` — baseline verification before and after probing
- `shadow-self-improve` — redteam findings feed into self-improvement proposals

## Crystallized From

- Pattern 22: Continuous Adversarial Hardening (SHADOW Patterns Library)
- Source: YouTube analysis 2026-05-03, "Nat Friedman and Daniel Gross with Collisons"
- Insight: occasional pen tests → continuous AI red-teaming, defense asymmetry
