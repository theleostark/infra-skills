---
name: 'SHADOW-X Live Theme Engine'
description: "Use when rendering adaptive live-context surfaces where theme follows state: work context, X/social intelligence, live scores/events, market signals, incident watch, or generative content boards. Use when deciding how to display live context as Calm, Watch, Focus, Urgent, Review, or Publish modes; when creating Craft cards, Markdown dashboards, HTML previews, Mermaid diagrams, action boards, or approval/publish surfaces from live events."
metadata:
  category: content/x
  family: shadow
  lifecycle: active
  canonical_slug: shadow-x-live-theme-engine
  icon_style: craft-category-glyph-v1
---

# Shadow-X Live Theme Engine

Use this skill to convert live context into the right **rendering mode**, **surface**, and **next-action UI**.

Core rule:

> Do not render everything. Render the state the operator needs to inhabit next.

## Workflow

1. Normalize input into a `LiveContextEvent` shape.
2. Classify semantic state:
   - domain: `work`, `x`, `live_score`, `market`, `incident`, `content`, `research`
   - urgency: `low`, `medium`, `high`, `critical`
   - visibility: `private`, `work`, `public`, `draft`
   - confidence and evidence quality
3. Pick render mode:
   - `Calm` — reflection, archive, pattern extraction
   - `Watch` — live feed, delta-first monitor
   - `Focus` — work execution, owner/action/evidence
   - `Urgent` — threshold crossed, escalation
   - `Review` — claim/evidence/risk before send/publish/relay
   - `Publish` — final content/action surface
4. Pick surface:
   - Craft card for quick glance
   - Markdown for durable notes/specs
   - Mermaid for causal/state diagrams
   - datatable/spreadsheet for rows
   - HTML preview for rich dashboards
   - PDF/PPTX/relay packet for work handoff
5. Render every output with this grammar:

```text
state + delta + evidence + risk + next action
```

## Mode selection heuristic

```python
if urgency == "critical":
    mode = "Urgent"
elif visibility in ["work", "public"] and requires_approval:
    mode = "Review"
elif domain in ["live_score", "market", "incident", "x"] and is_live:
    mode = "Watch"
elif domain == "content" and ready:
    mode = "Publish"
elif domain == "content":
    mode = "Review"
elif asks_for_reflection:
    mode = "Calm"
else:
    mode = "Focus"
```

## Output templates

### Craft/Markdown card

```markdown
## {mode}: {title}

**State:** ...  
**Delta:** ...  
**Evidence:** ...  
**Risk:** ...  
**Next action:** ...
```

### Content board card

Use when a live event can become content:

```json
{
  "status": "triaged",
  "channel": "x",
  "audience": "builders/operators",
  "hook": "...",
  "angle": "...",
  "claim": "...",
  "evidence_links": [],
  "draft": "...",
  "risk_flags": []
}
```

## Renderer script

Use the bundled renderer for deterministic HTML cards:

```bash
python3 scripts/render_live_theme.py examples/xynth-event.json -o /tmp/live-theme.html
```

Options:

- `--mode calm|watch|focus|urgent|review|publish` overrides automatic mode selection.
- Without `--mode`, the script chooses mode from `domain`, `urgency`, `visibility`, and `signals`.

## References

Read only when needed:

- `references/live-theme-engine.md` — full concept/spec.
- `references/live-context-event.schema.json` — normalized event schema.
- `references/content-card.schema.json` — generative content card schema.
- `assets/live-theme-engine-preview.html` — interactive ShadowTech preview template.
- `scripts/render_live_theme.py` — deterministic HTML renderer.
- `examples/xynth-event.json` — sample event.

## Guardrails

- Separate evidence from interpretation.
- Use `Review` mode before public/work/external outputs.
- Do not expose private implementation details in public/work artifacts.
- For market/financial signals, keep output as research/alert-only unless a separate compliant execution system exists.
