---
name: 'SHADOW Typora Engine'
description: "Use when controlling Typora themes on the fly, externalizing/reverse-engineering Typora CSS themes, injecting Shadow design systems into Typora, switching semantic render modes like calm/watch/focus/urgent/review/publish/work, or generating canonical shadow-typora theme overlays with naming manifests and AppleScript reloads."
metadata:
  category: system/typora
  family: shadow
  lifecycle: active
  canonical_slug: shadow-typora-engine
  icon_style: craft-category-glyph-v1
---

# Shadow Typora Engine

Use this skill when Typora should become a live rendering surface instead of a static Markdown editor.

Core model:

```text
base Typora theme + semantic injection overlay + document mode markers = live context surface
```

## Quick mode switch

Preferred control wrapper:

```bash
scripts/typora_shadowctl.sh list
scripts/typora_shadowctl.sh current
scripts/typora_shadowctl.sh set watch
scripts/typora_shadowctl.sh set review
scripts/typora_shadowctl.sh open /path/to/file.md review
scripts/typora_shadowctl.sh open-max /path/to/file.md review   # preferred: fast desktop-sized window
scripts/typora_shadowctl.sh open-wide /path/to/file.md work     # maximized + wide side-to-side document canvas
scripts/typora_shadowctl.sh open-full /path/to/file.md review  # native macOS fullscreen
scripts/typora_shadowctl.sh work-wide                          # make Astemo work canvas wide
scripts/typora_shadowctl.sh work-normal                        # restore normal Astemo document width
scripts/typora_shadowctl.sh infer /path/to/file.md
scripts/typora_shadowctl.sh auto /path/to/file.md   # maximized by default
scripts/typora_shadowctl.sh inventory
```

Official-cascade engine script (preferred for generated overlays):

```bash
scripts/typora-theme-engine-v3.sh watch night
scripts/typora-theme-engine-v3.sh review night
scripts/typora-theme-engine-v3.sh urgent night
scripts/typora-theme-engine-v3.sh work astemo
```

This writes `base.user.css` and `{theme}.user.css`, matching Typora's official CSS load order.

Legacy lower-level engine script:

```bash
scripts/typora-theme-engine-v2.sh watch Night
scripts/typora-theme-engine-v2.sh review Night
scripts/typora-theme-engine-v2.sh urgent Night
scripts/typora-theme-engine-v2.sh work Astemo
```

Install menu-friendly Typora aliases when needed:

```bash
scripts/install_typora_aliases.sh
```

This creates Typora menu themes: `Shadow Watch`, `Shadow Focus`, `Shadow Review`, `Shadow Urgent`, `Shadow Work`, and `Shadow Publish`. Typora may need a restart before new theme files appear in the Themes menu.

Supported modes:

- `calm`
- `watch`
- `focus`
- `urgent`
- `review`
- `publish`
- `work`

## Naming grammar

Use canonical IDs internally, but lowercase-hyphen filenames for Typora:

```text
Canonical ID: shadow-typora.<role>.<surface>.<mode>.<polarity>.<base>.<version>
Typora file:  shadow-typora-<role>-<surface>-<mode>-<polarity>-<base>-<version>.css
User overlay: {theme}.user.css
```

Expose short aliases to humans:

- `Shadow Watch`
- `Shadow Focus`
- `Shadow Review`
- `Shadow Urgent`
- `Shadow Work`
- `Shadow Publish`

## Workflow

1. Pick semantic state from context: Calm / Watch / Focus / Urgent / Review / Publish / Work.
2. Pick base theme: usually `Night`; use `Astemo` for work-branded docs, `Github`/`Whitey` for publish/light docs.
3. Generate canonical overlay CSS.
4. Copy overlay to Typora runtime import: `shadow-live-mode.css`.
5. Ensure base theme imports `shadow-live-mode.css`.
6. Use AppleScript/System Events to reload Typora's theme menu.

## References

Read as needed:

- `references/shadow-typora-engine.md` - architecture and mode strategy.
- `references/naming-schema.md` - canonical naming and manifest rules.
- `references/theme-manifest.json` - externalized theme metadata.
- `references/typora-official-theme-model.md` - official docs-grounded theme model.
- `references/naming-schema-v2.md` - corrected canonical ID vs Typora filename schema.
- `references/theme-gallery-taxonomy.md` - Typora gallery theme-family taxonomy.
- `scripts/typora_shadowctl.sh` - preferred wrapper for list/current/set/open/open-max/open-wide/open-full/work-wide/work-normal/infer/auto/inventory; `auto` opens maximized by default to avoid the small-window-to-fullscreen flash.
- `scripts/typora-theme-engine-v3.sh` - official `.user.css` cascade mode-switch engine.
- `scripts/typora-theme-engine-v2.sh` - legacy mode-switch engine.
- `scripts/install_typora_aliases.sh` - creates menu-friendly Shadow theme aliases.

## Guardrails

- Prefer `base.user.css` and `{theme}.user.css` over editing vendor theme files.
- Use lowercase-hyphen CSS filenames for any theme that must appear in Typora's menu.
- Backup base themes before first import injection if using legacy v2.
- Treat `externalized-themes/` snapshots as immutable source truth.
- Prefer generated overlays over editing vendor themes directly.
- Never put private context or secrets into CSS `content:` strings.

## Verifiability principle (SP-027)

CSS artifact modules must be visually verified, not assumed.

1. **CSS rules exist ≠ renders correctly.** A class in the `.css` file does not prove it visually works.
2. **Typora live preview does NOT render `<div class="...">` blocks.** All CSS artifact modules are export-only (HTML export, PDF export, browser).
3. **Verify before claiming.** Build a self-contained HTML file with the CSS, open in browser, screenshot every module, record pass/fail.
4. **Log evidence.** Screenshot filenames, what they show, which modules pass or fail.
5. **Fix before shipping.** If a module fails, fix the CSS and re-verify. Do not ship broken modules.

Verified module catalog: `Typora-Artifact-Verifiability-Report.md` (in project `04_Presentations/`).

Evidence strength hierarchy for CSS modules:
- ❌ "The class exists in the CSS file" — Level 1 (plausible)
- ⚠️ "I opened it in Typora" — unreliable (live preview doesn't render HTML divs)
- ✅ "Screenshot evidence from browser with the CSS loaded" — Level 3 (reproducible)
- ✅✅ "CI renders every module and compares to baseline" — Level 4 (independent)


## Pipeline

```
Intent → Resolve target → Execute operation → Verify result → Report
```

## Modes

| Mode | Output | When |
|------|--------|------|
| `default` | Standard output | Normal use |
| `verbose` | Detailed diagnostics | Debugging |
| `json` | Machine-readable output | Programmatic use |

## Artifact Routing

- Reports: `ShadowArchive/80-reports/`
- Config changes: `system/controls/` or `config/system/`

## Prerequisites

- SHADOW infrastructure accessible
- Appropriate auth/SSH for mesh targets

## Auto-Astemo Open (agent rule)

When opening ANY `.md` file in Typora from an agent session, use:

```bash
scripts/typora-open-astemo.sh <file.md> [wide|max|normal]
```

This auto-detects Astemo work files (by path pattern + content keywords) and forces the Astemo theme. Non-Astemo files fall through to `typora_shadowctl.sh auto` for mode inference.

**Do NOT use `open -a Typora <file>` directly** — always route through the auto-opener so the theme matches the work context.

Detection rules (priority order):
1. Path contains: `FH4S`, `XP7G`, `XY9H`, `FH4L`, `P703`, `Astemo_Context`, `astemo-ent`, `USGRP1`, `sduvvuru-qx`, `NAIPMLocalization` → **Astemo work mode**
2. Content contains: `category: enterprise/astemo`, `X10-02A`, `SASG`, `GCFT Leader`, `USGRP1` → **Astemo work mode**
3. Otherwise → **Infer via shadowctl** (review/watch/focus/calm/etc)

## Contract

- Never modify production config without operator confirmation
- Report all changes with before/after diff
- Preserve existing configurations with backup
- Always open `.md` files via `typora-open-astemo.sh` — never raw `open -a Typora`

