---
name: Friday Hud
description: Use when building F.R.I.D.A.Y.-branded interfaces, HUD-style dashboards, or monitoring UIs for the FRIDAY gateway device in the OpenClaw mesh
icon: icon.svg
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: friday-hud
  icon_style: craft-category-glyph-v1
---

# F.R.I.D.A.Y. HUD Design Pattern

## Overview
F.R.I.D.A.Y. (Female Replacement Intelligent Digital Assistant Youth) is the always-on Mac Mini gateway in the OpenClaw mesh. Its UI language is a dark military HUD aesthetic with warm orange/amber accents — matching the MCU AI assistant's actual visual identity.

Reports to: **Andy Stark** ("Boss")

## Lore Reference
- Voiced by Kerry Condon (Irish accent) in the MCU
- Replaced J.A.R.V.I.S. after Age of Ultron
- Addresses user as "Boss" (not "Sir" — that was J.A.R.V.I.S.)
- Direct, tactical, efficient personality — combat co-pilot, not butler
- J.A.R.V.I.S. = blue, F.R.I.D.A.Y. = orange (distinct visual identity)
- Reference image: concentric orange circles on dark background (targeting reticle motif)

## Design Tokens

```css
/* Core palette — warm dark */
--bg-0: #0a0d12;     /* deepest background */
--bg-1: #12161d;     /* card background */
--bg-2: #1a1610;     /* raised surfaces */
--border: #2a2218;   /* warm dark borders */
--text-0: #e8e0d8;   /* primary text (warm white) */
--text-1: #9a8e80;   /* secondary text */
--text-2: #5e5548;   /* muted text */
--orange: #ff8c00;   /* F.R.I.D.A.Y. orange — primary accent */
--orange-hover: #e07a00;  /* darker for hover/active */
--orange-dim: rgba(255, 140, 0, 0.12);  /* subtle backgrounds */
--green: #00ff88;    /* success / online */
--amber: #ffaa00;    /* warning */
--red: #ff4444;      /* error / critical */

/* Typography */
font-family: 'JetBrains Mono', 'SF Mono', 'Fira Code', monospace;
/* All headings uppercase, letter-spacing: 0.1em */
```

## Component Patterns

### Card
```
border-left: 3px solid var(--orange);
background: var(--bg-1);
border: 1px solid var(--border);
border-radius: 6px;
padding: 1.5rem;
```

### Status indicator (heartbeat dot)
```
8px circle, var(--green) with box-shadow glow
Animate: pulse 2s infinite (opacity 1 -> 0.4 -> 1)
```

### Navigation
```
Horizontal button bar, monospace uppercase
Active tab: color var(--orange), border-bottom 2px solid var(--orange)
Inactive: color var(--text-1), no border
```

### Section headers
```
Uppercase, monospace, color var(--orange), letter-spacing 0.1em
Thin 1px border-bottom in var(--border) below heading
```

### Logo / Branding
```css
/* Text-only, no images */
content: 'F.R.I.D.A.Y.';
font-family: 'JetBrains Mono', monospace;
font-size: 22px; font-weight: 700;
letter-spacing: 0.08em;
color: #ff8c00;
text-transform: uppercase;
animation: reactorPulse 3s ease infinite;

@keyframes reactorPulse {
  0%, 100% { text-shadow: 0 0 8px rgba(255, 140, 0, 0.4); }
  50% { text-shadow: 0 0 16px rgba(255, 140, 0, 0.7), 0 0 32px rgba(255, 140, 0, 0.3); }
}
```

### Badge / Pill
```css
font-size: 9px; font-weight: 700;
letter-spacing: 0.1em; text-transform: uppercase;
color: #ff8c00;
background: rgba(255, 140, 0, 0.1);
border: 1px solid rgba(255, 140, 0, 0.3);
border-radius: 10px;
padding: 2px 8px;
```

### Buttons
```css
background: #ff8c00;
color: #0a0d12;
font-weight: 600;
border: none;
/* Hover: background #e07a00 */
```

## Layout Rules
- Two-column grid on desktop: 2fr left (primary data), 1fr right (sidebar panels)
- Single column on mobile (stack vertically)
- Cards have 3px left border accent in var(--orange)
- Collapsible sections use chevron arrows, not +/- icons

## Branding
- Title: `F.R.I.D.A.Y.` (with periods, uppercase)
- Subtitle badge: small rounded pill (e.g., "GATEWAY", "MESH", "MCP")
- No logos or images — text-only identity
- Address user as "Boss" in any status messages
- Owner: Andy Stark

## Color Distinction from Other AIs
- J.A.R.V.I.S. = blue (#00d4ff) — the original, butler-like
- F.R.I.D.A.Y. = orange (#ff8c00) — the successor, tactical
- Karen = warm/friendly palette — Spider-Man's mentor
- E.D.I.T.H. = neutral — posthumous weapon system

## When NOT to Use
- ShadowTech branded interfaces (use shadowtech-design-system)
- Codex interfaces (use cc-design-system)
- GPT Codex interfaces (use gc-design-system)
- J.A.R.V.I.S.-branded interfaces (use blue palette, not this one)


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

## Contract

- Never modify production config without operator confirmation
- Report all changes with before/after diff
- Preserve existing configurations with backup

