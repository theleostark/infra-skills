# Shadow Typora Engine

## Essence

Typora can become a live rendering surface if we treat themes as composable layers:

```text
base Typora theme + semantic injection overlay + document mode markers = live context surface
```

The important discovery: **do not rewrite every Typora theme**. Use a stable base theme and inject state-specific design systems as small CSS overlays.

## Current theme map

- `night.css` — best base for Shadow because it includes dark Mermaid/code/source mode support.
- `shadow-x-inject.css` — live overlay injected into `night.css`.
- `shadow-x.css` — standalone Shadow-X theme.
- `astemo.css` — work/branded formal surface.
- `github.css` / `whitey.css` — neutral/light spec surfaces.
- `newsprint.css` / `pixyll.css` / `gothic.css` — editorial/reading surfaces.
- `kami.css` — dark focus writing surface.

## Engine architecture

```text
Typora theme directory
  ↓ externalize all themes
Theme inventory / token extraction
  ↓ classify theme family
Shadow token bridge
  ↓ generate overlay CSS
Inject overlay into active theme
  ↓ force Typora theme reload
Operator sees live state as design
```

## Runtime modes

| Mode | Base | Overlay | Use |
|---|---|---|---|
| Calm | newsprint/pixyll | soft archive overlay | reflection, essays |
| Watch | night | Shadow-X violet/cyan overlay | live context monitor |
| Focus | kami/night | action/evidence overlay | work execution |
| Urgent | night | red/amber escalation overlay | incidents, deadlines |
| Review | night/github | claim/evidence/risk overlay | before send/publish |
| Publish | github/whitey | clean final overlay | final readable copy |
| Work | astemo | work-safe overlay | Astemo/customer artifacts |

## Hot-swap strategy

Typora does not expose a first-class runtime theme API, but on macOS we can:

1. Write or update CSS under `~/Library/Application Support/abnerworks.Typora/themes/`.
2. Add an `@import` overlay to a selected base theme.
3. Use AppleScript/System Events to click `Themes → <theme>` and force reload.
4. For faster state changes, keep base theme constant (`Night`) and rewrite the imported overlay file.

## Design rule

The overlay file should be generated from a semantic state object, not hand-edited:

```json
{
  "mode": "watch",
  "accent": "violet",
  "density": "compact",
  "surface": "markdown-live-card",
  "emphasis": ["delta", "evidence", "next_action"]
}
```

## Safety

- Always backup a base theme before injecting.
- Prefer imported overlays over direct edits.
- Keep original externalized copies under `externalized-themes/`.
- Never put secrets or private context into CSS content strings.
