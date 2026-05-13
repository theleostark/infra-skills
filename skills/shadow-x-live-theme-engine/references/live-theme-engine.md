# Live Theme Engine — deeper rendering essence

## Essence

A **live theme engine** is not a skin. It is a rendering intelligence layer that watches live context and chooses the best surface, tone, density, urgency, and visual language for the moment.

```text
live context → semantic state → theme state → render surface → operator action
```

The core move is: **theme follows state**.

- Calm state → low contrast, sparse dashboard, reflective summaries.
- Urgent state → high contrast, alert rail, time-left, owner/action emphasis.
- Work state → evidence-first, conservative tone, relay-safe boundaries.
- X/social state → claim/evidence split, virality/context score, content hooks.
- Live score/event state → delta-first, clock/state prominence, next inflection.
- Generative state → board lanes, drafts, variants, review gates.

## Why this matters

Most dashboards render data. A live theme engine renders **attention**.

It answers:

1. What changed?
2. Why does it matter now?
3. What mode am I in?
4. What should I do next?
5. What should be hidden, softened, or escalated?

## Engine layers

### 1. Context ingestion

Sources become `LiveContextEvent` objects:

- X API / browser captures
- Outlook / Teams / Calendar
- score feeds / market feeds
- files / notes / manual observations
- agent outputs

### 2. Semantic classification

Each event gets classified into:

- domain: `work`, `x`, `live_score`, `market`, `incident`, `content`, `research`
- urgency: `low`, `medium`, `high`, `critical`
- confidence: `0–1`
- visibility: `private`, `work`, `public`, `draft`
- emotional/operational mode: `calm`, `watch`, `focus`, `urgent`, `review`, `publish`

### 3. Theme state

Theme state is a compact rendering contract:

```json
{
  "mode": "watch",
  "density": "compact",
  "tone": "analytical",
  "contrast": "medium",
  "motion": "pulse-low",
  "accent": "violet",
  "surface": "craft-card",
  "priority_slots": ["delta", "evidence", "next_action"]
}
```

### 4. Surface selection

The engine chooses output surface by job:

| Need | Surface |
|---|---|
| quick operator glance | Craft card |
| structured rows | datatable / spreadsheet |
| causal/path reasoning | Mermaid |
| high-fidelity review | HTML preview |
| work handoff | Markdown / PPTX / PDF packet |
| live monitoring | dashboard / ticker / board |
| creative iteration | generative content board |

### 5. Rendering grammar

Every render should expose:

```text
state + delta + evidence + risk + next action
```

A good card is not pretty first. It is **decision-shaped** first.

## Theme modes

### Calm

Use for summaries, reflection, end-of-day capture.

- Soft contrast
- No motion
- More context
- Fewer alerts
- Emphasis: learning, patterns, archive

### Watch

Use for live feeds where nothing is burning yet.

- Compact density
- Gentle pulse on changed items
- Delta column
- Staleness indicator
- Emphasis: thresholds, next check

### Focus

Use for work execution.

- Minimal decoration
- Owner/date/action prominence
- Evidence links
- Emphasis: next action and blockers

### Urgent

Use for critical live events, incidents, deadlines.

- High contrast
- Red/amber accent
- Countdown/clock
- Suppress nonessential context
- Emphasis: decision, owner, escalation

### Review

Use before publishing/sending/relaying.

- Evidence-first layout
- Claim/source separation
- Risk flags
- Human approval rail
- Emphasis: what could be wrong

### Publish

Use for ready content.

- Clean final preview
- Copy action
- Channel-specific constraints
- Provenance hidden unless needed
- Emphasis: final text and send boundary

## Adaptive rendering algorithm

```python
def render_live_context(event):
    semantic = classify_event(event)
    theme = choose_theme_state(semantic)
    surface = choose_surface(semantic, theme)
    model = build_view_model(event, semantic, theme)
    return render(surface, model, theme)
```

### Theme selection heuristic

```python
if semantic.urgency == "critical":
    mode = "urgent"
elif semantic.visibility in ["work", "public"] and semantic.requires_approval:
    mode = "review"
elif semantic.domain in ["live_score", "market", "incident"]:
    mode = "watch"
elif semantic.domain == "content":
    mode = "publish" if semantic.ready else "review"
else:
    mode = "focus"
```

## Rendering tokens

```json
{
  "colors": {
    "calm": "slate/blue",
    "watch": "violet/cyan",
    "focus": "blue/white",
    "urgent": "red/amber",
    "review": "amber/violet",
    "publish": "green/white"
  },
  "density": {
    "glance": "1 screen, 3 bullets max",
    "compact": "table/card hybrid",
    "deep": "sections + evidence + diagrams"
  },
  "motion": {
    "none": "static",
    "pulse-low": "changed item only",
    "pulse-high": "urgent badge/clock only"
  }
}
```

## Shadow-X application

The live theme engine sits above the adaptive workflows:

```text
Shadow-X router decides what this is.
Live Theme Engine decides how it should appear.
Generative board decides what it can become.
```

Examples:

| Context | Theme | Render |
|---|---|---|
| X product demo | Review | claim/evidence/gaps card |
| FH4S meeting action | Focus | owner/date/evidence action row |
| Sports score swing | Watch/Urgent | delta-first live score card |
| Market signal hit | Review | no-advice alert dashboard |
| Content idea from X | Publish/Review | hook variants + risk check |

## Operator-facing mantra

> Do not render everything. Render the state I need to inhabit next.

