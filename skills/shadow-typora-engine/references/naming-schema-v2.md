# Shadow Typora naming schema v2 — official-docs corrected

## Separation of concerns

There are two names:

1. **Canonical manifest ID** — rich, dot-delimited, machine-routable.
2. **Typora runtime filename** — lowercase hyphen slug, menu-safe.

## Canonical ID grammar

```text
shadow-typora.<role>.<surface>.<mode>.<polarity>.<base>.<version>
```

Example:

```text
shadow-typora.generated.review.review.dark.night.current
```

## Runtime filename grammar

```text
shadow-typora-<role>-<surface>-<mode>-<polarity>-<base>-<version>.css
```

Example:

```text
shadow-typora-generated-review-review-dark-night-current.css
```

## Menu alias filename grammar

For human menu use:

```text
shadow-<mode>.css
```

Examples:

```text
shadow-watch.css
shadow-review.css
shadow-urgent.css
shadow-work.css
```

## User overlay filename grammar

For official Typora cascade:

```text
{theme}.user.css
```

Examples:

```text
night.user.css
astemo.user.css
github.user.css
base.user.css
```

## Recommended runtime strategy

| Need | File |
|---|---|
| global Shadow vars across all themes | `base.user.css` |
| current Night mode overlay | `night.user.css` |
| current Astemo work overlay | `astemo.user.css` |
| menu-visible static alias | `shadow-review.css` |
| immutable externalized source | `externalized-themes/night.css` |
| provenance manifest | `vendor-map/theme-manifest.json` |

## Rule

Never use dot-delimited canonical IDs as Typora theme filenames. Keep dots in manifests only.
