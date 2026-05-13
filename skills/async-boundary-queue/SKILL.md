---
name: Async Boundary Queue
description: 'Use when the user needs: Delegate work across VPN, OS, device, or physical-access boundaries using shared queues like Reminders, Notes, and Dropbox without leaking local implementation details.'
icon: icon.svg
metadata:
  depends_on: [astemo-work-context,shadow-notes]
  category: enterprise/work
  family: enterprise
  lifecycle: active
  canonical_slug: async-boundary-queue
  icon_style: craft-category-glyph-v1
---


## Work context (read first — all dimensions)

Load **`astemo-work-context`**: `/Volumes/☯Duality/skills/astemo-work-context/SKILL.md` — resolves **program** (Dropbox roots, registry), **role** (which hat / audience), **org** (site, team, reporting), **task** (what artifact), and **workflow** (SASG / gates / sequence) before loading narrow `fh4s-*` or customer-only skills. Literal `FH4S_4E0A_IPM` / SHADOW paths below are **FH4S examples** unless this skill is explicitly multi-program.

# Async Boundary Queue

Agent-to-operator delegation through shared message surfaces that auto-sync across VPN, OS, and device boundaries. Enables an agent to request work that requires access the agent itself doesn't have (VPN, credentials, physical access).

## When to Use

- Agent needs something from a VPN-bound system (Laserfiche, company SharePoint, internal tools)
- Agent needs physical-world action (plug in device, restart server, read on-site label)
- Operator needs to leave instructions for the agent to execute later
- Cross-OS communication required (Mac agent ↔ WSL Ubuntu agent ↔ iPhone)
- Any scenario where request and response are separated by time, location, or access boundary

## Architecture

```
┌─────────────────────┐     icloud sync     ┌──────────────────┐
│  Agent (JARVIS/Mac) │ ◄──────────────────► │  WSL Ubuntu      │
│  - has shadow-* CLI │     Reminders/Notes    │  - has VPN        │
│  - no VPN access   │ ◄──────────────────► │  - on Windows host│
└──────────┬──────────┘     dropbox sync     │  - icloud.com     │
           │                                 └────────┬─────────┘
           │                                          │
           │  thumb drive (local)                      │
           ▼                                          ▼
    agent definition                            Laserfiche / SAP
    AGENTS.md / skills                         download → drop
    (detailed logic)                           confirm → remind
```

## Workflow

### Agent → Operator (requesting work)

1. Agent identifies a need for VPN-bound resource
2. Agent writes a **generic task** to the shared queue:
   - `shadow-remind add "Download 030E5-M6737-011 from Laserfiche" "Save rev C or later to Dropbox/_Incoming/Drawings/"`
   - `shadow-notes write "Drawings request" "Need lead frame 030E5-M6737-011. Check Laserfiche for latest rev."`
3. Task auto-syncs to iCloud → visible on operator's phone, work laptop, any browser
4. **No technique names, no pattern names, no "VPN-boundary" terminology** in the task

### Operator → Agent (completing work)

1. Operator opens icloud.com/reminders on work laptop (or phone Reminders app)
2. Operator sees the task, executes it (download from Laserfiche, save to Dropbox/_Incoming/)
3. Operator confirms by marking done or adding a note:
   - `shadow-remind note "030E5-M6737-011" "Downloaded rev C, saved to Dropbox/_Incoming/Drawings/"`
4. Next agent session: agent finds the file, processes it, marks task complete

### Agent → Operator (instructions for later)

1. Agent writes detailed instructions as a task:
   - `shadow-remind add "TR check: verify 3 change points before 4/20 meeting" "Check tracker rows 12, 18, 23. Confirm JP responses received. Flag any red status."`
2. Operator follows the instructions at their convenience

## Shared Message Surfaces

| Surface | CLI Tool | Accessible From | Best For |
|---------|----------|----------------|----------|
| Reminders (SD-agents) | `shadow-remind` | icloud.com, iOS, Mac | Tasks, to-dos, requests |
| Notes (Shared-Mac) | `shadow-notes` | icloud.com, iOS, Mac | Longer instructions, reference |
| Dropbox/_Incoming | File drop | Any synced device | File handoff (drawings, docs) |
| Dropbox/FH4S_4E0A_IPM | File drop | Any synced device | Astemo work file handoff |

## Dropbox Relay Upgrade

When Dropbox itself becomes the primary cross-device enterprise relay, do not
use it like a random file dump. Build a small control-plane structure around it.

### Preferred Layers

1. **Live project roots**
   - example: `FH4S_4E0A_IPM/`
   - active project artifacts live here
2. **Enterprise control plane**
   - example: `astemo-ent/`
   - overlays, shared kernel, reports, runtime/tooling manifests
3. **Legacy seed store**
   - example: `Astemo_Context/`
   - compatibility and seeded historical context only
4. **Relay buckets**
   - `_Incoming/`
   - `_Active/`
   - `_Archive/`

### Dropbox-as-Relay Rule

Use Dropbox for:

- durable context
- handoff prompts
- review packets
- enterprise overlays
- compact artifacts needed across macOS / WSL / browser-CLI boundaries

Do **not** use Dropbox for:

- `node_modules`
- virtual environments
- caches
- build output
- duplicated heavy repos

### Private Relay Model

When Dropbox is used as a relay between a trusted personal environment and a constrained external environment, treat the relay as a **private abstraction layer**, not as a mirror of the internal system.

#### Core rule

- trusted side = internal reference environment
- external side = constrained environment with different structure and limits
- relay = compact shared abstraction between them

Do **not** export the internal host structure, mesh topology, or implementation-specific paths into the relay.

Instead:

1. gather local structure and behavior **there**
2. write compact capture and gap artifacts into the relay
3. compare and analyze **here** on the trusted side

#### What the relay should carry

- neutral manifests
- source references, not sensitive host paths
- compact capture templates
- gap reports
- diff notes
- relay-safe wrappers and small CLIs

#### What the relay should not carry

- host topology dumps
- full internal filesystem maps
- direct mesh diagrams
- private runtime paths
- copied heavy tool trees

### Chrome-Core Heuristic

For constrained external environments, start with **Chrome as the core surface**.

Why:

- browser behavior is the fastest outside-in proxy for what the environment actually supports
- many adjacent surfaces depend on Chrome behavior anyway

Check in this order:

1. tabs and tab groups
2. group semantics (levels, colors, emoji, active pinning, stale quarantine)
3. bookmark organization and sync
4. browser CLI behavior (`chrome-web-cli` class surfaces)
5. bookmark/feed surfaces (`shadow-drop`, `shadow-feed`, Raindrop flows)

Then infer what adjacent tools must adapt to.

### Relay Packet Pattern

When a relay grows beyond a single handoff note, promote it into a **packet** with layered entrypoints rather than scattering ad hoc files.

#### Scrub-before-relay rule

Before copying local Shadow artifacts into Dropbox relay packets, classify them into:

- **share-safe** — compact, audience-facing summaries and generic diagrams
- **needs scrub** — useful, but contains local paths, hostnames, mesh labels, command surfaces, or implementation specifics
- **do not relay** — secrets, auth, tokens, raw vault data, full internal topology, peer SSH aliases, local absolute paths, or broad doctor/debug JSON

Default action:

1. scan packet candidates for local paths, hostnames, SSH aliases, provider keys, auth terms, and internal mesh labels
2. copy only share-safe files into Dropbox
3. generate sanitized summaries instead of copying raw doctor/debug JSON
4. if an unsafe packet was already placed in Dropbox, move it to local sealed storage and regenerate a scrubbed packet

#### Whole-surface scrub rule

Do not stop at the packet payload. For Dropbox/iCloud relay work, scrub the full active surface that makes the packet usable:

- payloads and returns
- packet maps, lifecycle files, and return contracts
- root/active couplers
- relay README and entrypoint docs
- small wrapper CLIs and guard-list code
- schemas and config files

The common failure is to sanitize the payload while leaking real identities through nearby docs, route labels, hardcoded validator paths, examples, or CLI blocklists. Specific identifier knowledge belongs in the trusted local deidentifier/enricher, not in synced relay tooling. Dropbox-visible tools should use neutral route labels and generic checks only.

For bidirectional A2A relay, use local sealed mapping:

- Dropbox-visible route labels: `gh-main`, `gh-enterprise-tooling`, `gh-work-artifacts`
- real account handles, host aliases, personal names, and re-identification maps: local sealed storage only
- validation: run both payload scan and whole-surface scan before declaring a packet safe

Local sealed retraction pattern:

```text
ShadowArchive/90-sensitive-sealed/dropbox-relay-retractions/<packet-id>-<timestamp>/
```

Use this when a packet was started in Dropbox and later found to expose internal-only detail. Do not delete the evidence outright; stop sync exposure by moving it out of Dropbox, then document the retraction and rebuild a sanitized packet.

#### Minimum packet layers

1. **Shortest human entry**
   - one compact summary (`executive-summary.md`)
2. **Practical human entry**
   - one operational entry packet (`start-here.md`)
3. **Machine entry map**
   - one compact JSON index (`packet-map.json`)
4. **Machine return contract**
   - one JSON contract defining required outputs (`return-contract.json`)
5. **Lifecycle tracker**
   - request / ack / return / compare progress (`lifecycle.json`)
6. **Trusted-side synthesis sheet**
   - a fixed `compare-here.md` surface where returned artifacts are analyzed

#### Root + Active dual coupler

For practical relay navigation, expose the packet in two places:

- **root coupler** — tells you the packet exists
  - example: `relay-packet.md`, `relay-packet.json`
- **active coupler** — tells you the current work state
  - example: `_Active/.../status-board.md`, `status-board.json`

This gives both people and agents a fast way to discover:

- what packet is active
- where the request lives
- where the returns go
- where synthesis happens

#### Acknowledgment rule

Before waiting for the full return packet, require a tiny early acknowledgment surface (`ack.md`).

Why:

- separates "not seen" from "in progress"
- reduces ambiguity in async loops
- makes lifecycle tracking more honest

#### Return discipline

Keep the return compact:

- local capture
- gap report
- diff note
- trusted-side compare sheet

Do not let the packet grow into a dump of raw environment data.

### Compliant Low-Noise External Probing

If the constrained environment is enterprise-managed:

- use manual or on-demand execution
- prefer read-only discovery first
- keep footprint small and explicit
- do not bypass, evade, or suppress enterprise security controls

Minimize scope through **smaller captures**, not through stealth.

### Cross-Device Agent Handoff File

When another agent on another machine should auto-pick up work, a root-level
handoff file is a valid async boundary primitive.

Good handoff file contents:

- purpose
- operator identity
- operating model
- exact roots and active file paths
- ownership boundaries
- current truth / current mistakes
- recommended next lanes
- deliverable locations
- storage constraints

This works especially well when:

- the second agent runs on Windows 11 + WSL Ubuntu
- enterprise web surfaces are accessed through browser / web CLI
- Dropbox is the only practical neutral sync surface

### Relay Agent Separation

Treat relay agents as boundary operators, not as product agents or tenant data
stores.

Use this layer split:

- `shadow` = product/runtime primitives and internal mesh behavior
- `shadow-ent` / `shadow-agent-ent` = tenant-neutral enterprise relay runtime
- tenant wrapper = Astemo/FH4S labels, program defaults, work-facing commands
- shared relay = scrubbed packets, acknowledgments, returns, neutral route labels
- local sealed side = real identities, account handles, host aliases, topology,
  OAuth authority details, and re-identification maps

#### Canonical placement

Reusable relay-agent rules and runtime docs belong under:

```text
shadow-agent-ent/runtime/a2a/
```

Tenant-specific wrappers may be named `astemo-*`, but they should call the
neutral relay runtime rather than duplicating it. Old ingested Dropbox mirrors
such as `ShadowArchive/20-ingested/dropbox/.../relay-agent.py` are evidence and
compatibility references, not the canonical runtime surface.

#### Naming rule

- Shadow product work: `Shadow | <runtime/task>`
- Enterprise runtime work: `shadow-ent | relay | <task>`
- Astemo tenant work: `Astemo <Program> | <task>`

Do not show `Shadow FH4S`, because that mixes product identity with a work
program. The correct visible form is `Astemo FH4S | <task>`.

#### Relay-agent behavior

A relay agent should:

1. receive a compact task or packet
2. validate that the visible packet is scrubbed
3. write or update `ack.md` / lifecycle state early
4. route by neutral labels only
5. keep raw captures and local enrichment out of synced relay surfaces
6. return compact results and gap notes, not full environment dumps
7. let the trusted side re-identify or synthesize locally

If a relay agent needs real names, accounts, hosts, or paths to operate, that
mapping belongs in local sealed storage, not in Dropbox or iCloud.

### Root Discipline

If Dropbox is acting as the relay layer, the root should stay nearly empty:

- project roots
- enterprise control-plane roots
- intake / active / archive buckets
- a root guide
- temporary handoff prompts

Loose work files at root are a routing failure unless they are intentionally
staged for immediate pickup.

## Information Boundary Rules

### LOCAL ONLY (never syncs to company-visible surfaces)

- Pattern knowledge and technique names
- AGENTS.md agent definitions
- ~/.agents/skills/ (all skill files)
- shadow-a2a/bin/ scripts (implementation)
- ShadowArchive/ internal structure

### SHARED (syncs via iCloud/Dropbox)

- Task descriptions in Reminders/Notes
- File handoff in Dropbox folders
- Task body text can reference specific files, part numbers, systems
- Task body text should be **generic and direct** — describes what, not how

### What NOT to put in shared queues

- Pattern names ("VPN-boundary delegation", "async queue", "cross-OS routing")
- Internal tool names beyond what operator already knows
- Infrastructure details (shadow-a2a, skill paths, AGENTS.md structure)
- Agent-to-agent communication mechanics

## Task Language Guide

**Do:**
- "Download drawing X from Laserfiche, save to Dropbox/_Incoming/"
- "Check rows 12-18 in the tracker, flag any red items before Thursday"
- "Run the weekly report script and save output to _Active/"

**Don't:**
- "Use the VPN-boundary delegation pattern to..."
- "The shadow-remind bridge will auto-route this across..."
- "Agent definition in AGENTS.md section 4 covers this..."

## Key Decisions

- **Reminders vs Notes for tasks**: Reminders for action items (can be marked done). Notes for reference/instructions (persist after reading).
- **Priority levels**: P0 = urgent/time-bound, P1 = important, P5 = when possible
- **SD-agents list**: The canonical agent task list. Don't create task lists in other Reminders lists — keeps things consolidated.
- **_Incoming folder**: Staging area for files being handed off. Agent processes and moves files out after ingestion.

## Gotchas

- AppleScript `body` property in Reminders rejects empty string — always check for empty body before setting
- Dropbox cloud-only files (0B on disk) will hang rsync — check file size before copying, skip 0-byte files
- iCloud Drive files get evicted to save space — accessing them triggers re-download (may be slow)
- Reminders fuzzy match (`contains`) can match wrong tasks — use specific enough names
- osascript heredoc expansion: always use `EOF` (not `'EOF'`) when shell variables need expansion into AppleScript

## Crystallized From

- Session: 2026-04-16 (iTerm2 profiles + iCloud bridge work)
- Original task: Building Mac app CLI bridges for cross-OS agent communication
- Pattern discovered: VPN-boundary delegation via shared message queues (Reminders + Notes as async queue, Dropbox as file handoff, iCloud as automatic sync transport)
- Operator insight: "this shouldn't be revealed to the company, so it should stay as some how can we do that?"
- Session: 2026-04-17 (Astemo Dropbox relay restructuring)
- Additional crystallization: Dropbox became the explicit enterprise relay layer between macOS, WSL Ubuntu on the work laptop, and browser/web-CLI M365 surfaces. Native Windows is host-only; context should stay in WSL and Dropbox. The reusable pattern is to add a small control-plane root (`astemo-ent`), keep live project roots intact, downscope legacy seed roots, and use a detailed root-level handoff file for async cross-device agent continuation.
- Session: 2026-04-17 (private relay + Chrome-core refinement)
- Additional crystallization: The relay should stay private and abstract rather than reflecting the internal Shadow mesh directly. The trusted personal side remains the analysis reference; the work laptop is treated as a constrained outside surface. Chrome is the first-class external probe because tab grouping, bookmarks, feed flow, and browser CLI behavior reveal the highest-leverage capability gaps. Capture there, compare here.
- Session: 2026-04-17 (relay packet completion)
- Additional crystallization: once the relay packet grew beyond one handoff note, the stable reusable structure became: shortest summary, practical start file, machine map, machine return contract, lifecycle tracker, acknowledgment file, trusted-side compare sheet, plus root and active dual couplers. This packet pattern keeps async loops legible for both humans and agents without exposing internal topology.
- Session: 2026-04-22 (Shadow / shadow-ent / Astemo separation and Kaku label leak)
- Additional crystallization: product, enterprise framework, and tenant program labels must stay separate. `Shadow FH4S` is a branding/data-boundary leak; the visible tenant form is `Astemo <Program> | <task>`. Relay-agent runtime belongs in tenant-neutral `shadow-agent-ent/runtime/a2a`, with Astemo wrappers only adding program defaults and work-facing labels.
