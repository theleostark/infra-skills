# Typora official theme model — reverse-engineered notes

Sources checked:

- `https://theme.typora.io/`
- `https://theme.typora.io/doc/Install-Theme/`
- `https://theme.typora.io/doc/Write-Custom-Theme/`
- `https://support.typora.io/About-Themes/`
- `https://support.typora.io/Add-Custom-CSS/`
- `https://support.typora.io/Code-Block-Styles/`
- `https://support.typora.io/Change-Styles-in-Focus-Mode/`
- `https://support.typora.io/Draw-Diagrams-With-Markdown/`
- `https://support.typora.io/Debug-Themes/`

## Critical correction

Typora theme **filenames** must be menu-safe:

```text
lowercase letters + hyphens; no capital letters; avoid whitespace/non-alpha chars
```

Typora maps filename to menu label:

```text
my-first-typora-theme.css → My First Typora Theme
shadow-watch.css → Shadow Watch
```

Therefore our canonical ID can remain rich in manifests, but **runtime CSS filenames should use lowercase hyphen slugs**, not dot-delimited IDs.

Bad as Typora menu file:

```text
shadow-typora.generated.live.watch.dark.night.current.css
```

Good as manifest ID, but runtime file should be:

```text
shadow-typora-generated-live-watch-dark-night-current.css
```

Or better for menu alias:

```text
shadow-watch.css
```

## Official CSS load order

Typora loads CSS in this order:

1. Typora base styles
2. Current theme CSS
3. `base.user.css`
4. `{current-theme}.user.css`

This is the clean injection point. Instead of editing `night.css`, prefer:

```text
night.user.css
```

for Night-only overlay, or:

```text
base.user.css
```

for global overlay across themes.

## Theme folder

macOS theme folder:

```text
~/Library/Application Support/abnerworks.Typora/themes/
```

New themes require Typora restart to appear in the Themes menu.

Existing theme overlays via `.user.css` can often be refreshed by re-selecting the theme, but restart is the reliable path.

## Official selector model

Typora is a WebKit/Chromium webview. Themes are CSS.

Important selectors / hooks:

| Target | Selector / variable |
|---|---|
| whole window | `html`, `body` |
| writing area | `#write` |
| headings | `h1`…`h6`, optionally `#write h1` |
| block type | `[mdtype="heading"]`, `[mdtype="table"]`, etc. |
| paragraph line | `.md-line` |
| code fence before CodeMirror | `pre.md-fences.mock-cm` |
| code fence | `pre.md-fences` |
| code syntax theme | `.cm-s-inner` |
| source mode | `.cm-s-typora-default` |
| YAML frontmatter | `pre.md-meta-block` |
| math block | `[mdtype="math_block"]`, `.mathjax-block` |
| TOC | `.md-toc` |
| task list | `ul.task-list li.task-list-item` |
| inline wrappers | `span[md-inline="..."]` |
| inline meta syntax | `.md-meta`, `.md-content`, `.md-expand` |
| focus mode body | `.on-focus-mode` |
| focused block | `.md-focus` |
| focused list container | `.md-focus-container` |

## CSS variables Typora expects

Typora recommends overriding variables for common colors/fonts:

```css
:root {
  --bg-color: #ffffff;
  --text-color: #333333;
  --md-char-color: #C7C5C5;
  --meta-content-color: #5b808d;
  --primary-color: #428bca;
  --primary-btn-border-color: #285e8e;
  --primary-btn-text-color: #fff;
  --window-border: 1px solid #eee;
  --active-file-bg-color: #eee;
  --active-file-text-color: inherit;
  --active-file-border-color: #777;
  --side-bar-bg-color: var(--bg-color);
  --item-hover-bg-color: rgba(229, 229, 229, 0.59);
  --item-hover-text-color: inherit;
  --monospace: monospace;
}
```

Shadow overlays should set these first, then add Shadow-specific styling.

## CodeMirror / code block rule

Typora code fences use:

```css
.cm-s-inner
```

Source code mode uses:

```css
.cm-s-typora-default
```

So a complete Shadow code layer must style both.

## Focus mode rule

When focus mode is enabled:

- `<body>` gets `.on-focus-mode`
- focused block gets `.md-focus`
- focused list containers get `.md-focus-container`

Simple official knob:

```css
:root { --blur-text-color: #FFF; }
```

Advanced overlays can dim non-focused blocks under `.on-focus-mode`.

## Mermaid / diagram variables

Typora supports Mermaid variables via CSS:

```css
:root {
  --mermaid-theme: dark; /* default, base, dark, forest, neutral, night */
  --mermaid-font-family: "JetBrains Mono", monospace;
  --mermaid-sequence-numbers: off;
  --mermaid-flowchart-curve: linear;
  --mermaid--gantt-left-padding: 75;
}
```

Diagram alignment:

```css
.md-diagram-panel-preview { text-align: left; }
```

## Better Shadow architecture after official docs

Old prototype:

```text
edit base theme → append @import shadow-live-mode.css
```

Better official architecture:

```text
immutable base theme
+ generated {theme}.user.css
+ optional base.user.css for global Shadow variables
```

This aligns with Typora’s intended cascade and avoids corrupting vendor themes.
