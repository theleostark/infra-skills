# Shadow Typora naming schema

## Problem

Typora theme names are currently human/theme labels (`night`, `github`, `kami`, `astemo`). They do not encode:

- base vs overlay vs generated runtime state
- light/dark polarity
- semantic mode
- intended surface/audience
- safety boundary
- version or provenance

For a live rendering engine, names must be machine-routable.

## Naming principle

A Typora theme artifact name should answer:

```text
Who owns it? What surface is it? What state does it render? What base does it patch? Is it safe to overwrite?
```

## Canonical grammar

```text
shadow-typora.<role>.<surface>.<mode>.<polarity>.<base>.<version>.css
```

Where:

| Segment | Meaning | Examples |
|---|---|---|
| `shadow-typora` | namespace | fixed |
| `role` | artifact role | `base`, `overlay`, `inject`, `generated`, `snapshot`, `vendor` |
| `surface` | intended use | `work`, `x`, `live`, `content`, `review`, `publish`, `read`, `code`, `brand` |
| `mode` | live theme state | `calm`, `watch`, `focus`, `urgent`, `review`, `publish`, `work`, `neutral` |
| `polarity` | visual polarity | `dark`, `light`, `print`, `auto` |
| `base` | patched base theme | `night`, `github`, `astemo`, `kami`, `none` |
| `version` | version/date | `v1`, `20260502`, `dev` |

## Examples

| Current | Better name | Why |
|---|---|---|
| `shadow-x-inject.css` | `shadow-typora.inject.live.watch.dark.night.v1.css` | an injected live/watch overlay for Night |
| `shadow-x.css` | `shadow-typora.base.live.watch.dark.none.v1.css` | standalone base theme |
| `shadow-live-mode.css` | `shadow-typora.generated.live.watch.dark.night.current.css` | runtime-generated overlay |
| `astemo.css` | `shadow-typora.vendor.work.work.light.astemo.snapshot.css` | externalized branded work theme |
| `night.css` | `shadow-typora.vendor.code.focus.dark.night.snapshot.css` | externalized dark code base |
| `github.css` | `shadow-typora.vendor.review.neutral.light.github.snapshot.css` | externalized neutral review base |

## Short aliases

Use aliases for Typora menu readability, but maintain manifest metadata.

| Alias | Canonical target |
|---|---|
| `Shadow Watch` | `shadow-typora.generated.live.watch.dark.night.current.css` |
| `Shadow Review` | `shadow-typora.generated.review.review.dark.night.current.css` |
| `Shadow Urgent` | `shadow-typora.generated.live.urgent.dark.night.current.css` |
| `Shadow Work` | `shadow-typora.generated.work.work.light.astemo.current.css` |
| `Shadow Publish` | `shadow-typora.generated.publish.publish.light.github.current.css` |

## Directory schema

```text
typora-shadow-engine/
├── externalized-themes/          # raw snapshots, never edited
├── vendor-map/                   # canonical metadata for existing themes
├── overlays/                     # reusable named overlays
├── generated/                    # runtime generated CSS
├── aliases/                      # menu-friendly symlinks/copies
├── engine/                       # scripts/specs
└── analysis/                     # reverse engineering outputs
```

## Manifest schema

Each theme artifact should have a manifest entry:

```json
{
  "id": "shadow-typora.inject.live.watch.dark.night.v1",
  "file": "overlays/shadow-typora.inject.live.watch.dark.night.v1.css",
  "role": "inject",
  "surface": "live",
  "mode": "watch",
  "polarity": "dark",
  "base": "night",
  "source": "generated-from-shadowtech-tokens",
  "mutable": true,
  "safe_to_overwrite": true,
  "typora_menu_alias": "Shadow Watch"
}
```

## Mode taxonomy

| Mode | Visual contract | Default base | Accent |
|---|---|---|---|
| `calm` | reflective, soft, archive | `newsprint` or `pixyll` | blue/slate |
| `watch` | live, delta-first, compact | `night` | violet/cyan |
| `focus` | work execution, action/evidence | `kami` or `night` | green/blue |
| `urgent` | escalation, high contrast | `night` | rose/amber |
| `review` | claim/evidence/risk approval | `night` or `github` | amber/violet |
| `publish` | clean final copy | `github` or `whitey` | green/neutral |
| `work` | enterprise/customer-safe | `astemo` | Astemo red |

## Mutability rule

| Role | Mutability | Back up? | Notes |
|---|---:|---:|---|
| `vendor` | immutable | yes | Externalized original theme |
| `base` | rare edits | yes | Standalone custom theme |
| `inject` | editable | yes before first install | Stable overlay |
| `generated` | overwrite freely | no | Runtime mode CSS |
| `snapshot` | immutable | yes | Captured state for provenance |
| `alias` | overwrite copy/symlink | no | Menu-friendly name |

## Recommendation

Use **canonical IDs in manifests and scripts**, but expose **short aliases in Typora menu**.

Typora menu should show:

- `Shadow Watch`
- `Shadow Focus`
- `Shadow Review`
- `Shadow Urgent`
- `Shadow Work`
- `Shadow Publish`

Engine internals should write:

- `shadow-typora.generated.live.watch.dark.night.current.css`
- `shadow-typora.generated.review.review.dark.night.current.css`
- etc.
