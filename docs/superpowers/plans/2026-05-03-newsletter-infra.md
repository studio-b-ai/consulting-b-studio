# Newsletter Infrastructure Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create three beehiiv publications (Dispatch, DLI, Markdown), configure DNS, brand subscribe pages, wire HubSpot integrations, and write welcome sequences.

**Architecture:** beehiiv Max account already active. Two publications are new (Dispatch, Markdown); one exists and migrates (DLI/SBDLI Brief). DNS lives in GoDaddy for the b.studio zone. HubSpot integration is beehiiv-native (no custom webhook needed). Newsletter infra ships before websites — pub IDs from this plan are required inputs for the website plan's beehiiv embeds.

**Tech Stack:** beehiiv MCP (`mcp.beehiiv.com/mcp`), GoDaddy DNS (manual — no API), HubSpot MCP (account-level), beehiiv dashboard (manual steps for branding + custom CSS)

**Dependency note:** The beehiiv MCP is wired in `.mcp.json`. Use it for all publication CRUD. DNS steps require GoDaddy dashboard access — they cannot be automated. Flag pub IDs to Kevin after Task 1 so he can add DNS records in parallel.

---

## File map

| File | Action | Purpose |
|---|---|---|
| `benchmarks-b-studio/index.html` | Modify | Swap DLI subscribe button URL from `brief.benchmarks.b.studio` → `dli.b.studio` |
| `docs/pub-ids.md` | Create | Record pub IDs captured from beehiiv API — shared input for website plan |

---

### Task 1: Create Dispatch and Markdown publications via beehiiv API

**Files:**
- Create: `docs/pub-ids.md`

- [ ] **Step 1: Create Dispatch publication**

Use beehiiv MCP tool `create_publication`:
```
name: "Dispatch, by Studio B"
description: "War stories and field notes from Studio B. For operators who want to know what's actually happening inside the firms that are getting AI to work — not what the consultants are selling."
```

Record the returned `id` — this is the Dispatch pub ID.

- [ ] **Step 2: Create Markdown publication**

Use beehiiv MCP tool `create_publication`:
```
name: "Markdown, by Studio B"
description: "How we build AI-native software on real operations — and what breaks. Behind-the-scenes of Bolt, AcuOps, and everything else Studio B ships."
```

Record the returned `id` — this is the Markdown pub ID.

- [ ] **Step 3: Record pub IDs**

Create `docs/pub-ids.md`:
```markdown
# beehiiv Publication IDs

| Newsletter | Pub ID | Domain |
|---|---|---|
| Dispatch, by Studio B | pub_XXXXXXXX-... | dispatch.b.studio |
| DLI, by Studio B | pub_e647b558-3a6e-4a29-87ff-c5fd2b566c35 | dli.b.studio |
| Markdown, by Studio B | pub_XXXXXXXX-... | markdown.b.studio |
```

Fill in actual pub IDs from Steps 1 and 2.

- [ ] **Step 4: Commit**

```bash
git add docs/pub-ids.md
git commit -m "feat(infra): record beehiiv pub IDs for all three publications"
```

- [ ] **Step 5: Share pub IDs with Kevin**

Post to Kevin via Slack: "Dispatch pub ID: [id] / Markdown pub ID: [id] / DLI pub ID: pub_e647b558-3a6e-4a29-87ff-c5fd2b566c35 — ready for DNS step."

---

### Task 2: DNS configuration (GoDaddy — manual)

**Files:** None (DNS is external config)

DNS is managed in GoDaddy for the `b.studio` zone. These steps require GoDaddy dashboard access. Kevin must perform these or grant access.

- [ ] **Step 1: Add Dispatch CNAME**

In GoDaddy DNS for `b.studio`:
- Type: CNAME
- Name: `dispatch`
- Value: `cname.beehiiv.com`
- TTL: 1 hour

- [ ] **Step 2: Add DLI CNAME**

In GoDaddy DNS for `b.studio`:
- Type: CNAME
- Name: `dli`
- Value: `cname.beehiiv.com`
- TTL: 1 hour

- [ ] **Step 3: Add Markdown CNAME**

In GoDaddy DNS for `b.studio`:
- Type: CNAME
- Name: `markdown`
- Value: `cname.beehiiv.com`
- TTL: 1 hour

- [ ] **Step 4: Verify propagation**

After adding CNAMEs (allow 5–30 min):
```bash
dig CNAME dispatch.b.studio +short
# Expected: cname.beehiiv.com.

dig CNAME dli.b.studio +short
# Expected: cname.beehiiv.com.

dig CNAME markdown.b.studio +short
# Expected: cname.beehiiv.com.
```

- [ ] **Step 5: Configure custom domains in beehiiv**

For each publication, use beehiiv dashboard (Settings → Custom Domain):
- Dispatch → `dispatch.b.studio`
- DLI (existing pub `pub_e647b558...`) → `dli.b.studio` (was `brief.benchmarks.b.studio`)
- Markdown → `markdown.b.studio`

beehiiv will show a TXT verification record — add it to GoDaddy alongside the CNAME.

- [ ] **Step 6: Verify custom domains show green in beehiiv**

In beehiiv dashboard for each publication: Settings → Custom Domain → status should show "Active" or green checkmark. If still pending after 30 min, check GoDaddy TTL and try again.

---

### Task 3: Brand Dispatch subscribe page

**Files:** None (beehiiv dashboard config — no code)

All styling is done in beehiiv dashboard → Publication Settings → Branding. Custom fonts require beehiiv Max (already active).

- [ ] **Step 1: Set Dispatch subscribe page branding**

In beehiiv dashboard → Dispatch publication → Settings → Branding:
- Publication name: `Dispatch, by Studio B`
- Background color: `#F7D344` (yolk)
- Button color: `#0D0D0D` (ink)
- Button text color: `#F7D344` (yolk)
- Subscribe page headline: `Dispatch, by Studio B`
- Subscribe page description: `War stories and field notes from Studio B. For operators who want to know what's actually happening inside the firms that are getting AI to work — not what the consultants are selling.`

- [ ] **Step 2: Add Fraunces 900 via custom CSS**

In beehiiv dashboard → Dispatch → Settings → Custom CSS, add:
```css
@import url('https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,900;1,9..144,900&display=swap');

.subscribe-page h1,
.subscribe-page .publication-name {
  font-family: 'Fraunces', serif;
  font-weight: 900;
}
```

- [ ] **Step 3: Verify subscribe page at dispatch.b.studio/subscribe**

Navigate to `https://dispatch.b.studio/subscribe` in browser. Confirm:
- Background is yolk `#F7D344`
- Button is ink `#0D0D0D`
- Fraunces displays on headline
- Description text is correct

---

### Task 4: Brand DLI subscribe page

- [ ] **Step 1: Set DLI subscribe page branding**

In beehiiv dashboard → DLI publication (`pub_e647b558...`) → Settings → Branding:
- Publication name: `DLI, by Studio B`
- Background color: `#F4F1EB` (paper)
- Button color: `#1A3328` (forest)
- Button text color: `#F4F1EB` (paper)
- Subscribe page headline: `DLI, by Studio B`
- Subscribe page description: `We're building the open lower middle market lending index — and publishing the journey as we go. Quarterly index data, market event commentary, and field notes from the construction. Ground truth, not surveys.`

- [ ] **Step 2: Add Fraunces 900 via custom CSS**

In beehiiv dashboard → DLI → Settings → Custom CSS:
```css
@import url('https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,900;1,9..144,900&display=swap');

.subscribe-page h1,
.subscribe-page .publication-name {
  font-family: 'Fraunces', serif;
  font-weight: 900;
}
```

- [ ] **Step 3: Verify subscribe page at dli.b.studio/subscribe**

Navigate to `https://dli.b.studio/subscribe`. Confirm:
- Background is paper `#F4F1EB`
- Button is forest `#1A3328`
- Title reads "DLI, by Studio B"

---

### Task 5: Brand Markdown subscribe page

- [ ] **Step 1: Set Markdown subscribe page branding**

In beehiiv dashboard → Markdown publication → Settings → Branding:
- Publication name: `Markdown, by Studio B`
- Background color: `#0D0D0D` (ink)
- Button color: `#D94425` (vermillion)
- Button text color: `#F4F1EB` (paper)
- Subscribe page headline: `Markdown, by Studio B`
- Subscribe page description: `How we build AI-native software on real operations — and what breaks. Behind-the-scenes of Bolt, AcuOps, and everything else Studio B ships.`

- [ ] **Step 2: Add JetBrains Mono via custom CSS**

In beehiiv dashboard → Markdown → Settings → Custom CSS:
```css
@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&display=swap');

.subscribe-page h1,
.subscribe-page .publication-name {
  font-family: 'JetBrains Mono', monospace;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.subscribe-page {
  color: #F4F1EB;
}
```

- [ ] **Step 3: Verify subscribe page at markdown.b.studio/subscribe**

Navigate to `https://markdown.b.studio/subscribe`. Confirm:
- Background is ink `#0D0D0D`
- Button is vermillion `#D94425`
- JetBrains Mono displays on headline
- Text is readable (paper `#F4F1EB` on ink bg)

---

### Task 6: Enable HubSpot integrations

**Files:** None (beehiiv + HubSpot dashboard config)

beehiiv has a native HubSpot integration. Enable it per publication. Kevin's HubSpot account ID: `49070660`.

- [ ] **Step 1: Connect HubSpot to Dispatch**

In beehiiv → Dispatch → Settings → Integrations → HubSpot:
- Connect with Kevin's HubSpot account (portal ID `49070660`)
- Enable: "Create contact on subscribe"
- Map field: set a static property `newsletter_source = "dispatch"` on contact creation

- [ ] **Step 2: Connect HubSpot to DLI**

In beehiiv → DLI → Settings → Integrations → HubSpot:
- Same account connection
- Map field: `newsletter_source = "dli"`

- [ ] **Step 3: Connect HubSpot to Markdown**

In beehiiv → Markdown → Settings → Integrations → HubSpot:
- Same account connection
- Map field: `newsletter_source = "build"` (HubSpot property value — maps to Build entity)

- [ ] **Step 4: Test each integration**

For each publication: subscribe with a test email (e.g. `kevin+test-dispatch@bibelhausen.com`).
Then in HubSpot MCP, search for that contact:
```
search_crm_objects: contacts where email = "kevin+test-dispatch@bibelhausen.com"
```
Confirm contact exists with `newsletter_source = "dispatch"`. Repeat for dli and build.

- [ ] **Step 5: Delete test contacts in HubSpot**

```
manage_crm_objects: delete contact [id from Step 4]
```
Repeat for all three test contacts.

---

### Task 7: Welcome sequences (all three publications)

**Files:** None (beehiiv dashboard)

Each publication gets a 3-email welcome sequence. Write actual copy below — do not leave placeholder text in beehiiv.

- [ ] **Step 1: Write and configure Dispatch welcome sequence**

In beehiiv → Dispatch → Automations → Welcome series. Create 3 emails:

**Email 1 — Immediate (on subscribe):**
Subject: `You're in.`
Body:
```
Welcome to Dispatch, by Studio B.

I'm Kevin. I run Studio B — a small team that builds and operates AI-native software on real companies. Heritage Fabrics ships on our WMS. Our AcuOps pipeline deploys Acumatica customizations. We built Amplify because we needed it.

Dispatch is where I write about what I'm seeing inside operators and PE-backed companies getting AI to actually work — not the pitch, the reality.

No cadence promise until I've shipped three issues and know the rhythm.

If you have a specific situation you're trying to figure out, reply. I read everything.

— Kevin
```

**Email 2 — Day 3:**
Subject: `What this is (and what it isn't)`
Body:
```
Dispatch isn't a newsletter about AI trends.

It's field notes. I write when something happens that's worth writing about — a $400K inventory discrepancy found three days into an audit, a deployment that took down a live warehouse for two hours, a PE sponsor who asked the wrong question at the wrong time.

The names are anonymized. The numbers are real.

If you want the background on Studio B: b.studio

If you want to talk about an engagement: consulting.b.studio

— Kevin
```

**Email 3 — Day 7:**
Subject: `One thing to know about how I work`
Body:
```
I don't take many engagements.

Not because I'm selective in the precious sense — because I'm running a portfolio of companies on the same software I sell. If I overcommit, something breaks. Usually something belonging to Heritage Fabrics.

That constraint makes me useful. I'm not selling you a methodology. I'm showing you what works in a real operation, then helping you run the same play.

If that's what you're looking for: consulting.b.studio

— Kevin
```

- [ ] **Step 2: Write and configure DLI welcome sequence**

In beehiiv → DLI → Automations → Welcome series:

**Email 1 — Immediate:**
Subject: `You're tracking the index.`
Body:
```
Welcome to DLI, by Studio B.

I'm Kevin Bibelhausen. I'm building a private credit fund focused on the global lower middle market — and the Studio B Direct Lending Index because the data didn't exist.

DLI is where I publish what I find: quarterly index data, market commentary when something moves, and field notes from the construction of the index itself. Ground truth, not surveys.

If you're an LP curious about the fund: capital.b.studio

— Kevin
```

**Email 2 — Day 3:**
Subject: `Why we built the index`
Body:
```
The LMM credit market has benchmarks. They cost a seat at the institutional table.

We're building the open alternative — constructed from observed deal data, not self-reporting. If you want to know what LMM direct lending actually costs today, you shouldn't need to pay Cliffwater $50K for the privilege.

That's what DLI, by Studio B is. And we publish the construction in real time — what we built, what broke, what the data is showing us.

— Kevin
```

**Email 3 — Day 7:**
Subject: `Cadence`
Body:
```
DLI publishes quarterly, anchored to the SBDLI index release.

Plus ad hoc when a market event or data anomaly warrants it. The Q1 2026 issue is coming. I'll write about what the spread data is telling us and what I think is driving it.

Until then: if you're looking at the LMM market and want to compare notes, reply.

— Kevin
```

- [ ] **Step 3: Write and configure Markdown welcome sequence**

In beehiiv → Markdown → Automations → Welcome series:

**Email 1 — Immediate:**
Subject: `You're reading the source.`
Body:
```
Welcome to Markdown, by Studio B.

I'm Kevin. I build Bolt (a WMS for continuous goods distributors), AcuOps (a DevOps platform for Acumatica), and Amplify (a content intelligence engine). They all run in production on real companies.

Markdown is where I write about building them — what shipped, what broke, why we made the call we made, what I'd do differently. Technical details welcome. Marketing language banned.

— Kevin
```

**Email 2 — Day 3:**
Subject: `What "build in public" means here`
Body:
```
It doesn't mean a tweet when we ship a feature.

It means writing about the two-day debugging session that found a bug in how Acumatica handles compound-key PUTs. The architecture decision we made and then reversed three weeks later. The production incident that took down a live warehouse.

If it's worth learning from, it's worth writing about. Including the parts that weren't our best work.

— Kevin
```

**Email 3 — Day 7:**
Subject: `The stack`
Body:
```
In case you're curious what we're building on:

Bolt is a WMS for continuous goods (fabric, cable, wire, tubing). It runs Heritage Fabrics. TypeScript, Railway, Postgres, Acumatica ERP write-back.

AcuOps is a DevOps platform for Acumatica VARs. CI/CD through the Customization API, 12 health probes, post-publish test suite. $300/mo flat.

Amplify is a content intelligence engine. It detects signals from your activity, drafts newsletter issues and social content in your voice, routes through Slack for approval. That's what sends this newsletter.

More at build.b.studio.

— Kevin
```

- [ ] **Step 4: Verify sequences are active**

In beehiiv → each publication → Automations: confirm all three welcome series show status "Active" not "Draft."

---

### Task 8: Update benchmarks.b.studio DLI subscribe button

**Files:**
- Modify: `benchmarks-b-studio/index.html`

The existing SBDLI Brief subscribe button on benchmarks.b.studio points to `brief.benchmarks.b.studio/subscribe`. It needs to point to `dli.b.studio/subscribe` now that the domain has migrated.

- [ ] **Step 1: Find the subscribe button in benchmarks-b-studio**

```bash
grep -n "brief.benchmarks\|beehiiv\|subscribe" /path/to/benchmarks-b-studio/index.html
```

Note the line number.

- [ ] **Step 2: Update the subscribe URL**

Find the subscribe button or link and change the href from:
`https://brief.benchmarks.b.studio/subscribe`
to:
`https://dli.b.studio/subscribe`

Also update any display text if it still says "SBDLI Brief" — change to "DLI, by Studio B."

- [ ] **Step 3: Verify in browser**

Open `benchmarks.b.studio` locally or on GitHub Pages. Click the subscribe button — confirm it navigates to `https://dli.b.studio/subscribe` and the DLI subscribe page loads correctly (paper background, forest button).

- [ ] **Step 4: Commit and push**

```bash
git add index.html
git commit -m "feat: update subscribe CTA from brief.benchmarks to dli.b.studio"
git push
```

- [ ] **Step 5: Verify on live site**

Wait 1–2 min for GitHub Pages to deploy. Navigate to `https://benchmarks.b.studio`, click subscribe button, confirm it reaches `https://dli.b.studio/subscribe`.

---

### Task 9: Configure beehiiv referral programs

**Files:** None (beehiiv dashboard)

beehiiv native referral program — subscribers earn rewards for referring others. Enable for Dispatch and DLI (highest value audiences). Markdown optional.

- [ ] **Step 1: Enable Dispatch referral program**

In beehiiv → Dispatch → Grow → Referral Program:
- Enable referral program
- Milestone 1 (1 referral): "Dispatch Archive Access" (free — just acknowledgment)
- Milestone 2 (3 referrals): Direct reply from Kevin (personal touch, manageable at early subscriber counts)
- Referral link auto-appended to each issue footer

- [ ] **Step 2: Enable DLI referral program**

In beehiiv → DLI → Grow → Referral Program:
- Enable referral program
- Milestone 1 (1 referral): "DLI Archive Access"
- Milestone 2 (3 referrals): Quarterly index data one week early

- [ ] **Step 3: Verify referral links appear in test issue**

Create a draft issue in each publication → preview → confirm referral link appears in footer with correct copy.

---

## Self-review

**Spec coverage check:**
- ✅ Create Dispatch publication — Task 1
- ✅ Create Markdown publication — Task 1
- ✅ DNS for all three domains — Task 2
- ✅ DLI domain migration from brief.benchmarks.b.studio — Task 2
- ✅ Dispatch subscribe page branding — Task 3
- ✅ DLI subscribe page branding — Task 4
- ✅ Markdown subscribe page branding — Task 5
- ✅ HubSpot integrations (all three) — Task 6
- ✅ Welcome sequences (all three) — Task 7
- ✅ benchmarks.b.studio DLI button update — Task 8
- ✅ Referral program — Task 9
- ✅ Newsletter XP best practices — covered across Tasks 3–5 (branding), 7 (welcome sequences), 9 (referral)

**Output required for website plan:** `docs/pub-ids.md` with all three pub IDs. Website plan cannot wire beehiiv embeds until this file exists.
