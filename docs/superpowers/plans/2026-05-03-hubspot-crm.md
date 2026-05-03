# HubSpot CRM Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Configure Studio B's HubSpot portal with three pipelines (consulting, capital, build), custom contact properties, HubSpot meeting scheduler, beehiiv→HubSpot subscriber sync per publication, and one reporting dashboard per entity.

**Architecture:** Kevin's HubSpot account hosts the Studio B portal (portalId `49070660`). All API calls go through the HubSpot MCP (already wired in `.mcp.json` at account level). The beehiiv native HubSpot integration is enabled per publication — subscribers auto-create contacts with `newsletter_source` set.

**Tech Stack:** HubSpot MCP (REST API via MCP tool), HubSpot Meetings Scheduler (enabled on Kevin's account), beehiiv native HubSpot integration (configured in beehiiv dashboard — no code required).

**Dependency:** Newsletter infrastructure plan must be complete before Task 4 (beehiiv→HubSpot integration) — you need the pub IDs.

---

## File Structure

No files created or modified. All configuration is done via HubSpot MCP API calls.

---

## Task 1: Create custom contact properties

**What:** Three custom properties on HubSpot contacts: `newsletter_source`, `subscriber_type`, `lp_inquiry_date`. These are the attribution backbone for the whole funnel.

- [ ] **Step 1: Check if properties already exist**

```
Use HubSpot MCP: search_properties
  objectType: "contacts"
  query: "newsletter_source"
```

Expected: empty results (not yet created). If any already exist, read their current definition and skip the creation step for that property.

- [ ] **Step 2: Create `newsletter_source` property**

```
Use HubSpot MCP: manage_crm_objects
  action: "create"
  objectType: "property"
  properties:
    name: "newsletter_source"
    label: "Newsletter Source"
    groupName: "contactinformation"
    type: "enumeration"
    fieldType: "select"
    description: "Which Studio B newsletter the contact first subscribed to"
    options:
      - label: "Dispatch, by Studio B"  value: "dispatch"
      - label: "DLI, by Studio B"       value: "dli"
      - label: "Markdown, by Studio B"  value: "build"
      - label: "Multiple"               value: "multiple"
```

Expected: property created with `name: "newsletter_source"`.

- [ ] **Step 3: Create `subscriber_type` property**

```
Use HubSpot MCP: manage_crm_objects
  action: "create"
  objectType: "property"
  properties:
    name: "subscriber_type"
    label: "Subscriber Type"
    groupName: "contactinformation"
    type: "enumeration"
    fieldType: "select"
    description: "What kind of relationship this contact is expected to have with Studio B"
    options:
      - label: "Consulting Prospect"  value: "consulting_prospect"
      - label: "LP Prospect"          value: "lp_prospect"
      - label: "Operator"             value: "operator"
      - label: "Both"                 value: "both"
```

- [ ] **Step 4: Create `lp_inquiry_date` property**

```
Use HubSpot MCP: manage_crm_objects
  action: "create"
  objectType: "property"
  properties:
    name: "lp_inquiry_date"
    label: "LP Inquiry Date"
    groupName: "contactinformation"
    type: "date"
    fieldType: "date"
    description: "Date the contact submitted an LP inquiry via capital.b.studio"
```

- [ ] **Step 5: Verify all three properties exist**

```
Use HubSpot MCP: search_properties
  objectType: "contacts"
  query: "newsletter_source"
```

Run separately for `subscriber_type` and `lp_inquiry_date`. Confirm all three return results.

- [ ] **Step 6: Record property IDs**

Print and record the internal IDs of all three properties. You'll need them if building workflows later.

---

## Task 2: Build Consulting pipeline

**What:** Five-stage pipeline tracking the journey from newsletter subscriber to signed engagement. Named "Studio B Consulting".

- [ ] **Step 1: Check if consulting pipeline already exists**

```
Use HubSpot MCP: get_crm_objects
  objectType: "pipelines"
  objectId: "deals"
```

Scan the results for a pipeline named "Studio B Consulting". If it exists, read its current stages and proceed to Step 4 (add missing stages).

- [ ] **Step 2: Create the Consulting pipeline**

```
Use HubSpot MCP: manage_crm_objects
  action: "create"
  objectType: "pipeline"
  properties:
    label: "Studio B Consulting"
    objectType: "deals"
    displayOrder: 1
```

Record the returned pipeline ID.

- [ ] **Step 3: Create pipeline stages in order**

Create each stage (use the pipeline ID from Step 2). Each stage needs a `label` and `displayOrder`:

Stage 1:
```
label: "Newsletter Subscriber"
displayOrder: 1
metadata:
  probability: 0.05
```

Stage 2:
```
label: "Engaged (clicked through to consulting.b.studio)"
displayOrder: 2
metadata:
  probability: 0.15
```

Stage 3:
```
label: "Meeting Booked"
displayOrder: 3
metadata:
  probability: 0.35
```

Stage 4:
```
label: "Proposal Sent"
displayOrder: 4
metadata:
  probability: 0.60
```

Stage 5:
```
label: "Engagement Signed"
displayOrder: 5
metadata:
  isClosed: true
  probability: 1.0
```

- [ ] **Step 4: Verify pipeline**

```
Use HubSpot MCP: get_crm_objects
  objectType: "pipelines"
  objectId: "deals"
```

Confirm "Studio B Consulting" pipeline appears with all 5 stages.

---

## Task 3: Build Capital pipeline

**What:** Five-stage pipeline tracking from newsletter subscriber to committed LP. Named "Studio B Capital".

- [ ] **Step 1: Create the Capital pipeline**

```
Use HubSpot MCP: manage_crm_objects
  action: "create"
  objectType: "pipeline"
  properties:
    label: "Studio B Capital"
    objectType: "deals"
    displayOrder: 2
```

Record the returned pipeline ID.

- [ ] **Step 2: Create pipeline stages**

Stage 1:
```
label: "DLI Subscriber"
displayOrder: 1
metadata:
  probability: 0.05
```

Stage 2:
```
label: "Engaged (clicked through to capital.b.studio)"
displayOrder: 2
metadata:
  probability: 0.15
```

Stage 3:
```
label: "LP Inquiry Received"
displayOrder: 3
metadata:
  probability: 0.30
```

Stage 4:
```
label: "Introductory Call Completed"
displayOrder: 4
metadata:
  probability: 0.50
```

Stage 5:
```
label: "Materials Sent"
displayOrder: 5
metadata:
  probability: 0.65
```

Stage 6:
```
label: "Committed LP"
displayOrder: 6
metadata:
  isClosed: true
  probability: 1.0
```

- [ ] **Step 3: Verify pipeline**

```
Use HubSpot MCP: get_crm_objects
  objectType: "pipelines"
  objectId: "deals"
```

Confirm "Studio B Capital" pipeline appears with all 6 stages.

---

## Task 4: Build Build pipeline

**What:** Four-stage pipeline for the build/product audience — Markdown subscribers converting to product trials, consulting engagements, or Amplify pipeline referrals.

- [ ] **Step 1: Create the Build pipeline**

```
Use HubSpot MCP: manage_crm_objects
  action: "create"
  objectType: "pipeline"
  properties:
    label: "Studio B Build"
    objectType: "deals"
    displayOrder: 3
```

- [ ] **Step 2: Create pipeline stages**

Stage 1:
```
label: "Markdown Subscriber"
displayOrder: 1
metadata:
  probability: 0.05
```

Stage 2:
```
label: "Engaged (clicked through to build.b.studio)"
displayOrder: 2
metadata:
  probability: 0.15
```

Stage 3:
```
label: "Product Trial or Meeting Requested"
displayOrder: 3
metadata:
  probability: 0.40
```

Stage 4:
```
label: "Converted (trial, engagement, or Amplify referral)"
displayOrder: 4
metadata:
  isClosed: true
  probability: 1.0
```

- [ ] **Step 3: Verify pipeline**

Confirm "Studio B Build" pipeline appears in deals pipelines list.

---

## Task 5: Configure HubSpot Meetings scheduler links

**What:** Confirm the two meeting scheduler links are live and correctly configured — one for consulting (Kevin's primary calendar), one for Capital LP inquiries.

- [ ] **Step 1: Verify Consulting meeting link**

Open in browser: `https://meetings.hubspot.com/kbibelhausen`

Confirm:
- Page loads with Kevin's name
- 30-minute slot is bookable
- Calendar availability is accurate
- Meeting is associated with Kevin's HubSpot account

If the link 404s, go to HubSpot → Sales → Meetings → Create meeting link.

- [ ] **Step 2: Verify Capital LP meeting link**

Open in browser: `https://meetings.hubspot.com/bibelhausen/studio-b-capital-lp-inquiry`

Confirm page loads. If it 404s, create via HubSpot → Sales → Meetings → Create meeting link, with slug `studio-b-capital-lp-inquiry`.

- [ ] **Step 3: Confirm meeting links are pipeline-connected**

In HubSpot Meetings settings, each meeting type should be set to auto-create a deal in the correct pipeline on booking:
- `kbibelhausen` → Studio B Consulting pipeline, stage "Meeting Booked"
- `studio-b-capital-lp-inquiry` → Studio B Capital pipeline, stage "Introductory Call Completed"

Configure via HubSpot dashboard → Meetings → [meeting name] → Settings → "Create deal for each meeting booked".

- [ ] **Step 4: Record confirmed meeting links**

Write confirmed links to `docs/meeting-links.md`:

```markdown
# HubSpot Meeting Links

- **Consulting (30min):** https://meetings.hubspot.com/kbibelhausen
- **Capital LP inquiry:** https://meetings.hubspot.com/bibelhausen/studio-b-capital-lp-inquiry
```

```bash
cd /Users/kevin/dev/consulting-b-studio
git add docs/meeting-links.md
git commit -m "docs: confirmed HubSpot meeting links"
```

---

## Task 6: Wire beehiiv → HubSpot subscriber sync

**What:** Enable the native beehiiv HubSpot integration on each publication so new subscribers auto-create HubSpot contacts with `newsletter_source` set.

This is configured in the beehiiv dashboard, not via API. Steps are manual in beehiiv UI.

**Prerequisites:**
- All three publications exist in beehiiv (from newsletter infra plan)
- `newsletter_source` property exists in HubSpot (Task 1 above)

- [ ] **Step 1: Connect beehiiv to HubSpot (one-time OAuth)**

In beehiiv: Settings → Integrations → HubSpot → Connect → Authorize with Kevin's HubSpot account.

This creates a single OAuth connection reused by all three publications.

- [ ] **Step 2: Enable HubSpot integration for Dispatch**

In beehiiv: Select "Dispatch, by Studio B" publication → Settings → Integrations → HubSpot → Enable.

Configure field mapping:
```
beehiiv subscriber email  → HubSpot email
beehiiv subscriber name   → HubSpot first name / last name
(static value)            → HubSpot newsletter_source = "dispatch"
```

Test: Subscribe to Dispatch with a test email. Confirm a HubSpot contact is created within 5 minutes with `newsletter_source = "dispatch"`.

- [ ] **Step 3: Enable HubSpot integration for DLI**

In beehiiv: Select "DLI, by Studio B" publication → Settings → Integrations → HubSpot → Enable.

Field mapping: same as Dispatch but `newsletter_source = "dli"`.

Test: Subscribe to DLI with a second test email. Confirm HubSpot contact created with `newsletter_source = "dli"`.

- [ ] **Step 4: Enable HubSpot integration for Markdown**

In beehiiv: Select "Markdown, by Studio B" publication → Settings → Integrations → HubSpot → Enable.

Field mapping: same as above but `newsletter_source = "build"`.

Test: Subscribe with a third test email. Confirm HubSpot contact created with `newsletter_source = "build"`.

- [ ] **Step 5: Verify three test contacts in HubSpot**

```
Use HubSpot MCP: search_crm_objects
  objectType: "contacts"
  filterGroups:
    - filters:
        - propertyName: "newsletter_source"
          operator: "HAS_PROPERTY"
```

Confirm all three test contacts appear with correct `newsletter_source` values.

---

## Task 7: Build reporting dashboards

**What:** One HubSpot dashboard per entity, each tracking the key metrics for that pipeline. Done in HubSpot UI — dashboard creation is not well-supported via the CRM API.

- [ ] **Step 1: Create Consulting dashboard**

In HubSpot: Reports → Dashboards → Create dashboard → "Studio B Consulting"

Add these reports (use HubSpot's built-in report builder for each):

1. **Subscriber count by newsletter** — Contact property: `newsletter_source`, breakdown by value, last 30 days
2. **Dispatch open rate** — Pull from beehiiv (manual entry or beehiiv→HubSpot metric sync when available)
3. **Meetings booked (last 30 days)** — Activity report: meetings, date range 30 days
4. **Meetings booked (last 90 days)** — Same, 90 days
5. **Pipeline value by newsletter source** — Deals in "Studio B Consulting" pipeline, grouped by `newsletter_source`

- [ ] **Step 2: Create Capital dashboard**

In HubSpot: Reports → Dashboards → Create dashboard → "Studio B Capital"

Add these reports:

1. **DLI subscriber count** — Contacts with `newsletter_source = "dli"`, all time
2. **LP inquiry count (last 90 days)** — Contacts with `lp_inquiry_date` set, last 90 days
3. **Capital pipeline stages** — Deals in "Studio B Capital" pipeline, by stage
4. **Materials sent count** — Deals in "Materials Sent" stage, last 90 days

- [ ] **Step 3: Verify dashboards load**

Navigate to each dashboard. Confirm reports render (they may show zero data — that's expected before subscribers arrive). Confirm no "no data available" errors that indicate misconfigured reports.

- [ ] **Step 4: Note dashboard URLs**

Save the HubSpot dashboard URLs for both:
```
Consulting: https://app.hubspot.com/reports-dashboard/49070660/view/[dashboard-id]
Capital: https://app.hubspot.com/reports-dashboard/49070660/view/[dashboard-id]
```

Write to `docs/hubspot-dashboards.md`:

```markdown
# HubSpot Dashboards

Portal ID: 49070660

- **Consulting:** [URL after creation]
- **Capital:** [URL after creation]
```

```bash
cd /Users/kevin/dev/consulting-b-studio
git add docs/hubspot-dashboards.md
git commit -m "docs: HubSpot dashboard URLs"
```

---

## Self-Review

**Spec coverage check:**
- ✅ `newsletter_source` property (Task 1) — Dispatch / DLI / Markdown / Multiple
- ✅ `subscriber_type` property (Task 1)
- ✅ `lp_inquiry_date` property (Task 1)
- ✅ Consulting pipeline — 5 stages per spec §4.2 (Task 2)
- ✅ Capital pipeline — 6 stages per spec §4.3 (Task 3)
- ✅ Build pipeline — 4 stages (Task 4)
- ✅ HubSpot meeting scheduler — both links verified (Task 5)
- ✅ beehiiv → HubSpot subscriber sync — all three publications (Task 6)
- ✅ Reporting dashboards — one per entity (Task 7)
- ✅ UTM tracking (spec §4.4) — no task needed; UTM parameters auto-captured by HubSpot when contact visits consulting.b.studio or capital.b.studio via newsletter links. Verify UTM is enabled in HubSpot Settings → Tracking → UTM Parameters after deployment.

**What is NOT in this plan (out of scope):**
- HubSpot Solutions Partner application (§7.1) — separate track, not a blocker
- HubSpot Operations Agent (§7.2) — AcuDev extension track, not this plan
- Affiliate registration for HubSpot (§7.3) — separate track
