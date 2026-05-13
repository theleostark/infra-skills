---
name: SHADOW Lists
description: 'Use when creating, reading, updating, or syncing Microsoft Lists (SharePoint lists) as the always-on agent cloud state layer. Covers list provisioning, schema definition, CRUD operations, webhook subscriptions, Excel bidirectional sync, and agent task queue management. Trigger on "lists", "microsoft lists", "sharepoint list", "agent queue", "cloud tracker", "list sync", "create list", "update list", "list webhook", or any task needing always-on structured cloud state for agents.'
icon: icon.svg
metadata:
  category: enterprise/astemo
  family: shadow-lists
  lifecycle: active
  canonical_slug: shadow-lists
  icon_style: craft-category-glyph-v1
  replaces: smartsheet-single-program
  sp_number: SP-016
---

# shadow-lists — Always-On Agent Cloud Layer via Microsoft Lists

Microsoft Lists as structured, always-on, webhook-capable cloud state layer for SHADOW agents. Same Graph/REST API auth as existing M365 headless skills. No new billing.

## Architecture

```
Agent (pi / OpenCode)
  │  POST/PATCH/GET via SP REST API (browser SSO)
  ▼
Microsoft Lists (cloud-native structured data)
  │  One-click Excel export / live ODC connection
  ▼
Excel (human layer — JP counterparts, managers, reviews)
```

## Auth Model

**Primary path: Craft browser `evaluate`** — use the in-app browser to make SP REST API calls directly. The browser carries the M365 SSO cookies.

**Key requirement: Form Digest** — all POST/PATCH/DELETE operations need an `X-RequestDigest` token. GET operations do not.

```javascript
// Get form digest (required before any write operation)
(async () => {
  const dig = await (await fetch('/sites/USGRPxEV/_api/contextinfo', {
    method: 'POST',
    headers: { Accept: 'application/json;odata=verbose' }
  })).json();
  const token = dig.d.GetContextWebInformation.FormDigestValue;
  // Use token in X-RequestDigest header for all subsequent POSTs
})()
```

```javascript
// GET — no digest needed
(async () => {
  const r = await fetch("/sites/USGRPxEV/_api/web/lists/getbytitle('SHADOW-Agent-Tasks')/items?$top=50&$select=Id,Title,Status,Priority", {
    headers: { Accept: 'application/json;odata=nometadata' }
  });
  const d = await r.json();
  return d.value;
})()
```

```javascript
// POST (create) — needs digest
(async () => {
  const dig = await (await fetch('/sites/USGRPxEV/_api/contextinfo', {
    method: 'POST', headers: { Accept: 'application/json;odata=verbose' }
  })).json();
  const token = dig.d.GetContextWebInformation.FormDigestValue;
  const r = await fetch("/sites/USGRPxEV/_api/web/lists/getbytitle('SHADOW-Agent-Tasks')/items", {
    method: 'POST',
    headers: {
      Accept: 'application/json;odata=verbose',
      'Content-Type': 'application/json;odata=verbose',
      'X-RequestDigest': token
    },
    body: JSON.stringify({ __metadata: { type: entityType }, Title: '...', Status: 'queued' })
  });
})()
```

**Fallback path: Chrome CDP** — when Craft browser auth expires, use Chrome with `--remote-debugging-port=9222` (see `samples/cdp-test.cjs`). Requires manual MFA completion.

### Bitwarden + Passkeys (future)

Bitwarden CLI (`bw`) can store and autofill credentials for M365 login. With passkeys enabled on the Astemo Azure AD tenant, the auth flow becomes:
1. `bw get item "astemo-m365"` — extract username/password
2. Fill login form via browser automation
3. Passkey/WebAuthn prompt — requires hardware key or platform authenticator
4. Session valid for ~8 hours

This eliminates the manual MFA step for headless automation. Requires:
- Bitwarden CLI installed and unlocked (`bw unlock`)
- Passkey registered in Azure AD for the account
- Hardware authenticator available (YubiKey, Touch ID)

Not yet implemented — tracked as next improvement.

## API Surface (SharePoint REST)

| Operation | Method | Endpoint Pattern |
|-----------|--------|------------------|
| List all lists | GET | `{site}/_api/web/lists?$filter=Hidden eq false` |
| Get list items | GET | `{site}/_api/web/lists/getbytitle('{name}')/items` |
| Create item | POST | `{site}/_api/web/lists/getbytitle('{name}')/items` |
| Update item | POST | `{site}/_api/web/lists/getbytitle('{name}')/items({id})` + `X-HTTP-Method: MERGE` + `If-Match: *` |
| Delete item | POST | `{site}/_api/web/lists/getbytitle('{name}')/items({id})` + `X-HTTP-Method: DELETE` |
| Get columns | GET | `{site}/_api/web/lists/getbytitle('{name}')/fields?$filter=Hidden eq false` |
| Create column | POST | `{site}/_api/web/lists/getbytitle('{name}')/fields` |
| Create list | POST | `{site}/_api/web/lists` |
| List changes (delta) | GET | `{site}/_api/web/lists/getbytitle('{name}')/getchanges(...)` |
| Webhook register | POST | `{site}/_api/web/lists/getbytitle('{name}')/subscriptions` |

## Live Lists (provisioned and populated)

| List | Items | Entity Type | Purpose |
|------|-------|-------------|----------|
| SHADOW-Agent-Tasks | 4 | `SP.Data.SHADOWAgentTasksListItem` | Cross-program agent task queue |
| FH4S-Change-Points | 53 | `SP.Data.FH4SChangePointsListItem` | FH4S IPM tracker mirror (migrated from XLSX) |
| SHADOW-Programs | 9 | `SP.Data.SHADOWProgramsListItem` | Live program registry (replaces program-registry.json) |

All on site: `https://astemogroup.sharepoint.com/sites/USGRPxEV`

### FH4S-Change-Points data health
```
Total: 53 items (from FH4S_4E0A_IPM_Change_Point_Tracker.xlsx)
Risk:  H=8  M=18  L=27
Region: JP=49  US=4
Apps:  Common=48  Both=2  VarSpec=2  TBD=1
Source: ~/Library/CloudStorage/Dropbox/FH4S_4E0A_IPM/02_Change_Point_Tracker/
```

### SHADOW-Programs data
```
9 programs: 5 active_dev + 1 new + 1 cancelled + 1 pre_dev + 1 active_dev(Ford)
OEMs: Honda(7) + Ford(3)
FH4S has Smartsheet bind (2231837406482308) + Dropbox path
```

### SHADOW-Agent-Tasks (task queue)
Cross-program agent task queue. Webhook-driven dispatch.

| Column | Type | Notes |
|--------|------|-------|
| Title | Text | Task description |
| Program | Choice | FH4S, XP7G, XY9H, FH4L, Cross-Program |
| TaskType | Choice | triage, research, draft, review, ship, sync |
| Status | Choice | queued, in_progress, blocked, done, cancelled |
| Priority | Choice | P0, P1, P2, P3 |
| AssignedAgent | Text | Session ID or agent name |
| Source | Choice | email, teams, meeting, manual, webhook, transcript |
| SourceRef | Text | URL / message ID / thread ID |
| DueDate | DateTime | Target completion |
| Output | Text | Result URL or summary |
| Owner | Text | Human owner name |

### FH4S-Change-Points (tracker mirror)
Mirrors the 47-column XLSX tracker schema. Bidirectional Excel sync.

| Column | Type | Maps to XLSX col | Notes |
|--------|------|-------------------|-------|
| ItemNumber | Number | A | 1-57 |
| L3PartNumber | Text | B | Astemo part number |
| PartName | Text | C | English name |
| PartNameJP | Text | D | Japanese name |
| IsChangePoint | Choice (Yes/No) | E | CP flag |
| CPDescription | MultiLine | F | What changed |
| RiskLevel | Choice (H/M/L) | G | Risk assessment |
| Status | Choice | H | Not Started / Open / In Progress / Finalized / Complete |
| LeadRegion | Choice (JP/US) | L | Lead region |
| KASupplier | Text | M | Japan supplier |
| IAPSupplier | Text | N | US supplier |
| SupplierChange | Choice (Yes/No) | O | Supplier change flag |
| DrawingNumber | Text | P | Drawing ref |
| DrawingRev | Text | Q | Revision |
| Applicability | Choice (2WD/AWD/Both) | S | 2WD/AWD |
| USDENotes | MultiLine | W | US DE notes |
| JPDENotes | MultiLine | X | JP DE notes |
| TargetDate | DateTime | AA | Target date |
| HondaMLC | Choice (Yes/No) | — | MLC flag |

### SHADOW-Programs (live registry)
Replaces `program-registry.json` with live cloud state.

| Column | Type | Notes |
|--------|------|-------|
| ProgramID | Text | fh4s, xp7g, xy9h, fh4l |
| DisplayName | Text | Full display name |
| OEM | Choice | Honda, Ford, Nissan |
| Status | Choice | active_dev, ramp_up, production, sunset |
| DropboxPath | Text | Confirmed local Dropbox root |
| SmartsheetID | Text | Migration path (nullable) |
| LeadDE | Text | Lead engineer name |
| Site | Choice | USGRP1, Tarboro, JP |
| AutomationLevel | Choice | full, semi, manual |

## CRUD Operations

### Read all items
```javascript
const items = await page.evaluate(async (siteUrl, listName) => {
  const r = await fetch(
    `${siteUrl}/_api/web/lists/getbytitle('${listName}')/items?$top=500&$select=Id,Title,Status,Priority,Program,Modified,Created`,
    { headers: { 'Accept': 'application/json;odata=nometadata' } }
  );
  const data = await r.json();
  return data.value;
}, siteUrl, listName);
```

### Create item
```javascript
const newItem = await page.evaluate(async (siteUrl, listName, payload) => {
  const body = {
    '__metadata': { 'type': `SP.Data.${listName}ListItem` },
    ...payload
  };
  const r = await fetch(
    `${siteUrl}/_api/web/lists/getbytitle('${listName}')/items`,
    {
      method: 'POST',
      headers: {
        'Accept': 'application/json;odata=nometadata',
        'Content-Type': 'application/json;odata=verbose'
      },
      body: JSON.stringify(body)
    }
  );
  return await r.json();
}, siteUrl, listName, { Title: 'New task', Status: 'queued', Priority: 'P2' });
```

### Update item
```javascript
await page.evaluate(async (siteUrl, listName, itemId, updates) => {
  // First get the item to capture its type and etag
  const getResp = await fetch(
    `${siteUrl}/_api/web/lists/getbytitle('${listName}')/items(${itemId})`,
    { headers: { 'Accept': 'application/json;odata=verbose' } }
  );
  const item = await getResp.json();
  
  const r = await fetch(
    `${siteUrl}/_api/web/lists/getbytitle('${listName}')/items(${itemId})`,
    {
      method: 'POST',
      headers: {
        'Accept': 'application/json;odata=nometadata',
        'Content-Type': 'application/json;odata=verbose',
        'X-HTTP-Method': 'MERGE',
        'If-Match': item.d.__metadata.etag
      },
      body: JSON.stringify({
        '__metadata': { 'type': item.d.__metadata.type },
        ...updates
      })
    }
  );
}, siteUrl, listName, 1, { Status: 'in_progress' });
```

### Query with filter
```javascript
const critical = await page.evaluate(async (siteUrl, listName) => {
  const r = await fetch(
    `${siteUrl}/_api/web/lists/getbytitle('${listName}')/items?$filter=Priority eq 'P0' and Status ne 'done'&$orderby=Modified desc&$top=50`,
    { headers: { 'Accept': 'application/json;odata=nometadata' } }
  );
  return (await r.json()).value;
}, siteUrl, listName);
```

## Provisioning a New List

### Step 1: Create the list
```javascript
await page.evaluate(async (siteUrl) => {
  await fetch(`${siteUrl}/_api/web/lists`, {
    method: 'POST',
    headers: {
      'Accept': 'application/json;odata=verbose',
      'Content-Type': 'application/json;odata=verbose'
    },
    body: JSON.stringify({
      '__metadata': { 'type': 'SP.List' },
      'AllowContentTypes': true,
      'BaseTemplate': 100,
      'ContentTypesEnabled': true,
      'Title': 'SHADOW-Agent-Tasks',
      'Description': 'Agent task queue for SHADOW automation'
    })
  });
}, siteUrl);
```

### Step 2: Add columns
```javascript
// Text column
await fetch(`${siteUrl}/_api/web/lists/getbytitle('SHADOW-Agent-Tasks')/fields`, {
  method: 'POST',
  headers: {
    'Accept': 'application/json;odata=verbose',
    'Content-Type': 'application/json;odata=verbose'
  },
  body: JSON.stringify({
    '__metadata': { 'type': 'SP.FieldText' },
    'FieldTypeKind': 2,
    'Title': 'Program',
    'Required': true
  })
});

// Choice column
await fetch(`${siteUrl}/_api/web/lists/getbytitle('SHADOW-Agent-Tasks')/fields`, {
  method: 'POST',
  headers: {
    'Accept': 'application/json;odata=verbose',
    'Content-Type': 'application/json;odata=verbose'
  },
  body: JSON.stringify({
    '__metadata': { 'type': 'SP.FieldChoice' },
    'FieldTypeKind': 6,
    'Title': 'Status',
    'Choices': { '__metadata': { 'type': 'Collection(Edm.String)' }, 'results': ['queued', 'in_progress', 'blocked', 'done', 'cancelled'] }
  })
});
```

## Webhook Registration (for agent dispatch)

SharePoint list webhooks fire on item create/update/delete. Requires a public HTTPS endpoint.

```javascript
// Register webhook
const webhook = await fetch(
  `${siteUrl}/_api/web/lists/getbytitle('SHADOW-Agent-Tasks')/subscriptions`,
  {
    method: 'POST',
    headers: {
      'Accept': 'application/json;odata=verbose',
      'Content-Type': 'application/json;odata=verbose'
    },
    body: JSON.stringify({
      'resource': listId,
      'notificationUrl': 'https://shadow-gate.shadowlab.cc/api/lists-webhook',
      'expirationDateTime': new Date(Date.now() + 180 * 86400000).toISOString(), // 180 days max
      'clientState': 'shadow-webhook-secret'
    })
  }
);
```

**Webhook payload** (POST to notificationUrl):
```json
{
  "value": [{
    "subscriptionId": "guid",
    "clientState": "shadow-webhook-secret",
    "resource": "guid",
    "tenantId": "guid",
    "siteUrl": "/sites/...",
    "webId": "guid"
  }]
}
```

After receiving webhook → GET `/changes` to find what changed → react.

## Excel Bidirectional Sync

### Export (Lists → Excel)
1. In Lists UI: "Export to Excel" → downloads `.odc` file
2. Opens Excel with live connection to the list
3. Refresh to get latest data
4. Format for JP review / customer presentations

### Live Connection
The `.odc` connection string can be reused:
- Power Query: `SharePoint.Tables("https://astemogroup.sharepoint.com/sites/{site}", [ApiVersion=15])`
- Excel → Data → Get Data → From SharePoint List

### Import (Excel → Lists)
For initial migration or bulk updates:
1. Parse Excel with `openpyxl`
2. POST each row via REST API
3. Or use "Import from Excel" in Lists UI for quick one-shot

## Known Sites for List Placement

| Site | URL | Best For |
|------|-----|----------|
| USGRPxEV | `/sites/USGRPxEV` | INV DE team — agent tasks, cross-program |
| NAGEN4VAFH4S | `/sites/NAGEN4VAFH4S` | FH4S program — change point tracker |
| NAIPMLocalization | `/sites/NAIPMLocalization` | IPM localization — cross-program IPM |
| Hoe-Axle | `/sites/HoXP7Ge-AxleAM` | XP7G program |

## Migration from Current Systems

| From | To | Steps |
|------|-----|-------|
| Local XLSX tracker | FH4S-Change-Points list | Parse XLSX → bulk POST via REST API |
| `program-registry.json` | SHADOW-Programs list | Read JSON → POST each row |
| Smartsheet DE Activity | List + Excel sync | Phase out Smartsheet API dependency |
| A2A Dropbox handoff folders | Agent task queue | Create tasks instead of file drops |

## Constraints

- **No separate API token** — auth is browser SSO (same as all astemo-hl skills)
- **MFA required for initial auth** — user must log in manually once
- **Webhook needs public endpoint** — `shadow-gate` on AURION or Cloudflare Worker
- **List item limit** — 30M items per list (effectively unlimited for this use case)
- **Column limit** — ~500 columns per list (more than enough for 47-col tracker)
- **Choice field values** — must be predefined; adding new choices requires API call
- **Rate limits** — throttling after ~100 requests/min (not a concern for agent-scale usage)

## Integration with SHADOW

- **shadow-anchor**: After session anchor, write summary row to SHADOW-Agent-Tasks (output column)
- **shadow-mind**: Read active task queue as part of context assembly
- **meeting-check**: After meeting capture, create follow-up tasks in the queue
- **fh4s-change-point-update**: Update FH4S-Change-Points list instead of (or in addition to) local XLSX
- **TRMNL**: Render live list stats on e-paper display
- **shadow-gate**: Webhook receiver endpoint for real-time agent dispatch


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

