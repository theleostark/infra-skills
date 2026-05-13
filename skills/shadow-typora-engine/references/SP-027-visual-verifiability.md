# SP-027: Visual Verifiability for Rendered Artifacts

**Status:** crystallized  
**Date:** 2026-05-03  
**Origin:** FH4S localization Typora artifact module verification failure  
**Namespace:** principle (not patent)

---

## Essence

CSS rules exist ≠ renders correctly. Every visual artifact module must be verified with screenshot evidence before claiming it works.

## Problem it solves

Agents write CSS class names into Markdown, claim the "modules work," and ship without ever visually verifying that the classes render. This produces:
- Showcase documents that look broken when opened
- Unverified CSS accumulating in themes
- False confidence in rendering capabilities
- Users (operators) seeing broken output and losing trust

## The principle

1. **Build the artifact** — write the CSS, write the HTML/Markdown
2. **Render it** — load the CSS + HTML in a real browser (not just "the file exists")
3. **Screenshot it** — capture visual evidence of every module
4. **Log pass/fail** — record which modules work and which don't
5. **Fix before shipping** — if it fails, fix the CSS and re-verify
6. **Ship with evidence** — the report is part of the artifact, not optional

## Evidence strength hierarchy

| Claim | Strength |
|-------|----------|
| "The class exists in the CSS file" | Level 1 — plausible |
| "I opened it in Typora" | Unreliable — live preview strips HTML divs |
| "Screenshot evidence from browser with the CSS loaded" | Level 3 — reproducible |
| "CI renders every module and compares to baseline" | Level 4 — independent |

## Typora-specific constraint

Typora live preview does NOT render `<div class="custom-class">` blocks. All CSS artifact modules are **export-only** (HTML export, PDF export, browser rendering). Documents must remain readable in plain Markdown without the CSS.

## Where this lives

| Location | What it records |
|----------|----------------|
| `reference/VERIFIABILITY.md` | Claim decomposition row, anti-patterns, evidence strength entry |
| `shadow-typora-engine/SKILL.md` | Verifiability principle section with 5 rules |
| `astemo.css` header comment | Inline reminder in the CSS file itself |
| `Typora-Artifact-Verifiability-Report.md` | Per-module pass/fail with screenshot evidence |

## Collision note

SP-016 was initially (incorrectly) assigned to this principle on 2026-05-03. SP-016 is already allocated to "Firmware Context OS" (shadow-intel, shadow-lists). Corrected to SP-026. SP-026 was also already allocated to "Agentic Public-Domain Patent Mining and Commercial Viability Scoring" (sp025-sp026-candidate-routing.md, sp024-filing-readiness.md). Corrected again to SP-027. No patent registry entry — this is a principle, not a patentable invention. Lesson: always grep the full ecosystem before assigning SP numbers.

---

*Crystallized from a real failure: wrote 12 CSS module categories, claimed they all worked, shipped without visual verification. The operator caught it immediately.*
