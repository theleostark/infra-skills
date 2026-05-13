---
name: SHADOW Clone
description: Use when cloning and modify root-owned or signed local apps without damaging the installed original. Use when patching an installed macOS app, Electron bundle, language overlay, local reverse-engineered UI, or vendor app that should remain recoverable.
icon: icon.svg
metadata:
  category: automation/browser
  family: shadow
  lifecycle: active
  canonical_slug: shadow-clone
  icon_style: craft-category-glyph-v1
---

# Shadow Clone

Clone a local app into a controlled, reversible surface before patching it. Use when the original app is root-owned, App Store signed, vendor-updated, or otherwise unsafe to edit directly.

## When To Use

- The operator asks to reverse engineer, patch, translate, theme, or modify an installed macOS app.
- The app lives in `/Applications` and is owned by `root:wheel`.
- The app is Electron, Chromium, or web-bundle based and the useful patch target is under `Contents/Resources`.
- A signed app may reject direct resource edits, ad hoc re-signing, or modified `Info.plist`.

## Workflow

1. Identify the app and bundle:
   - `mdfind "kMDItemKind == 'Application' && kMDItemFSName == '*NAME*'"`
   - `plutil -p "/Applications/App.app/Contents/Info.plist"`
   - `find "/Applications/App.app/Contents/Resources" -maxdepth 5 -type f`
2. Fingerprint before touching:
   - ownership: `ls -ldO App.app`
   - signing: `codesign --verify --deep --strict --verbose=2 App.app`
   - resource bundle path and version.
3. Do not edit the original first. Create a clone:
   - `ditto "/Applications/App.app" "$HOME/Applications/AppShadowClone.app"`
   - `chmod -R u+w "$HOME/Applications/AppShadowClone.app"`
4. Patch only the narrow resource surface:
   - Prefer adding an overlay file plus one entrypoint reference.
   - Avoid changing `Info.plist` unless the app still launches after a resource-only clone.
   - For language patches, generate a dictionary from extracted UI strings, then inject a DOM/attribute observer that skips editable/user-content areas.
5. Verify launch behavior in this order:
   - Launch resource-only clone unchanged except resources.
   - If it fails signature validation, try ad hoc re-signing the clone: `codesign --force --deep --sign -`.
   - If old Electron crashes after ad hoc signing, preserve the signed original shell and try runtime override only if it does not flip the app into a dev environment.
   - If runtime override changes app mode, stop and mark that path unsuitable.
6. Keep fallback and restore paths visible:
   - Original app path.
   - Clone path.
   - Backups/hashes of edited entrypoints.
   - Exact launch command or wrapper.

## App Language Overlay Pattern

For Electron apps with hardcoded strings:

1. Extract strings from bundled JS with a streaming quote scanner; avoid broad regex over minified bundles.
2. Use an LLM to turn the inventory into a conservative dictionary.
3. Inject a runtime overlay:
   - translate exact UI labels and common generated phrases,
   - translate `title`, `placeholder`, `aria-label`, and button `value`,
   - skip `textarea`, `input`, `[contenteditable=true]`, Draft.js roots, and note/content containers.
4. Verify with screenshot or CDP. Never assume the overlay works because files were patched.

## Gotchas

- `/Applications` apps are often `root:wheel`; without passwordless sudo, patch a user clone or stage files for an admin prompt.
- Editing a signed app can make `codesign --verify` report `sealed resource is missing or invalid`.
- Editing `Info.plist` is higher risk than editing web resources; it can cause LaunchServices or AMFI failure.
- Ad hoc re-signing can make old Electron/App Store apps crash even when `codesign --verify` passes.
- `ELECTRON_START_URL` can force Electron apps into development mode; if app code switches API hosts or assumptions in dev mode, that route is not equivalent to patching the production bundle.
- Paths with spaces can break old Electron `file://` startup paths; prefer no-space runtime paths for experiments.
- Do not translate user content. Exact-match UI dictionaries are safer than broad Chinese-to-English DOM replacement.

## Crystallized From

- Session: 2026-04-22
- Original task: Masterway Note / `大师笔记.app` English localization on JARVIS.
- Pattern extracted: clone signed app first, patch resource overlay, verify signature and launch separately, and treat old Electron dev-mode overrides as a risky fallback rather than a normal solution.


## Pipeline

```
Intent → Resolve target app/tab → Execute browser action → Extract result → Return structured output
```

1. Identify target (URL, app, tab, element)
2. Connect to browser via Playwright/browser_tool or AppleScript
3. Execute action (navigate, extract, interact, screenshot)
4. Return structured output

## Modes

| Mode | Output | When |
|------|--------|------|
| `extract` (default) | Data from browser | Read operations |
| `interact` | Perform browser action | Click, fill, automate |
| `debug` | DOM/state inspection | Troubleshooting browser apps |

## Fallback Chain

1. browser_tool (Craft Agents built-in)
2. Playwright MCP tools
3. AppleScript Chrome/Safari control
4. chrome-web-cli

## Prerequisites

- Browser with authenticated session
- Playwright MCP or browser_tool available

## Contract

- **Never enter credentials** — assume session exists
- **Never submit forms** without operator approval
- **Respect existing tabs** — do not close operator tabs without permission

