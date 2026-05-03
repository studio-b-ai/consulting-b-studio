# Studio B Marketing Stack — Design Spec

**Date:** 2026-05-02  
**Scope:** Full newsletter/social-first marketing stack for all Studio B service brands: Consulting, Capital, and Build — plus Kevin's personal brand at bibelhausen.com. Covers social presence, newsletter infrastructure, website layer, CRM + attribution, and Amplify content engine (including Amplify-Content dialogue mode + podcast production).  
**Repos affected:** `consulting-b-studio`, `capital-b-studio`, `build-b-studio` (new), `b-studio-website` (hub redesign), `bibelhausen-com` (new), `amplify` (voice configs + signal sources)  
**Out of scope:** Bolt and AcuOps product marketing (separate motion). Amplify Console UI (M2, not yet built). Automated beehiiv→HubSpot content publishing bridge. SBDLI data pipeline changes (wasala repo).

**Meta-note:** This stack is the Amplify reference client implementation. Studio B is dogfooding the product it sells. Every design decision here is also a product decision — the onboarding pattern, the content workflow, the editorial loop, the podcast stack. Build it right for Studio B; resell it as Amplify.

---

## Section 0: Architecture

### 0.1 Entity map

Three service brands, one personal brand, one principal, one content engine:

| Entity | Domain | Newsletter | Audience | Conversion goal |
|---|---|---|---|---|
| Studio B (hub) | `b.studio` | — | All of the above | Route to the right lane |
| Consulting, by Studio B | `consulting.b.studio` | Dispatch, by Studio B | Operators, PE/PE-backed companies, fractional CTO prospects | Calendly booked → engagement signed |
| Capital, by Studio B | `capital.b.studio` | DLI, by Studio B | LPs, credit professionals, LMM operators | LP inquiry → introductory call |
| Build, by Studio B | `build.b.studio` | Build, by Studio B | Technical founders, Acumatica ecosystem, operators who build | Product trial / engagement / Amplify pipeline |
| Kevin Bibelhausen (personal) | `bibelhausen.com` | — | Anyone who's heard Kevin's name | Understand the full picture; route to the right entity |

`b.studio` is the firm hub — Kevin's social presence points here, and from here visitors self-select their lane. Each lane has its own site, newsletter, and conversion motion.

`bibelhausen.com` is Kevin as a person, not Kevin at Studio B. It covers all ventures, speaking, investment/angel activity. It does not have its own newsletter — subscribe CTAs point to Dispatch and/or DLI. Visual identity is distinct from the Studio B Billboard DNA; designed separately.

Kevin's personal brand (X.com + LinkedIn) is the connective tissue. He doesn't speak for three separate entities — he's one person building all of them, and the newsletters are dispatches from that work.

### 0.2 The funnel

```
Kevin posts on X.com / LinkedIn
        ↓
Amplify detects signal → drafts newsletter issue + social content
        ↓
Nael edits in beehiiv → Kevin approves via Slack → issue sends
        ↓
Subscribers click through to consulting.b.studio or capital.b.studio
        ↓
HubSpot tracks: subscriber → engagement → Calendly booked or LP inquiry
        ↓
Consulting client signed or LP committed
        ↓
Social proof → sells Bolt + AcuOps downstream
```

### 0.3 Tool stack (the growth stack)

| Layer | Tool | Role |
|---|---|---|
| Social | X.com, LinkedIn | Top-of-funnel signal + audience building |
| Newsletter | beehiiv (Max) | Issue drafting, subscriber management, custom domains |
| Content engine | Amplify | Signal detection → full-issue drafts + social post drafts |
| Dialogue / creation | Amplify-Content (Claude skill → M2 Console) | Kevin-initiated content sessions; podcast script generation |
| Podcast | Transistor.fm | RSS + Apple Podcasts + Spotify distribution |
| Podcast editing | Descript | AI-powered audio editing; one-click producer track removal |
| Social scheduling | Typefully | X.com thread scheduling from Amplify-Content output |
| CRM | HubSpot | Subscriber → lead → client attribution for all pipelines |
| AI voice | ElevenLabs | Producer voice for podcast; iterates privately before shipping |
| Websites | GitHub Pages | Conversion layer for all entities |
| Editorial | Nael (director of marketing) | Voice calibration, final edit before Kevin approves |

---

## Section 1: Social Presence

Kevin's personal accounts are the primary distribution channel. The newsletters are where relationships deepen; social is where they start.

### 1.1 Kevin's X.com profile

X.com is the primary signal source for Amplify and the primary audience-building surface for both entities.

**Profile configuration:**
- Bio: operator voice, one line positioning — references Studio B, what he builds, and what he measures. Points to `b.studio` as the primary link so visitors self-select their lane.
- Link in bio: `b.studio`
- Pinned post: latest Dispatch issue or a high-signal standalone thread (updated each issue)

**Posting cadence:**
- Daily or near-daily. Short-form versions of the war stories and observations that become Dispatch issues.
- Amplify drafts X.com threads as a by-product of drafting newsletter issues — same signal, two outputs.
- DLI/capital content lives on Kevin's personal account, not a separate Capital account. Kevin happens to be building a lending index. He talks about it.

### 1.2 Kevin's LinkedIn profile

LinkedIn is secondary but important for LP and PE audience reach. Same voice, slightly longer form acceptable.

**Profile configuration:**
- Headline: reflects both Consulting, by Studio B and Capital, by Studio B roles
- Featured section: links to both newsletters (`dispatch.b.studio/subscribe` + `DLI.b.studio/subscribe`)
- About section: written in Kevin's voice (not a resume summary — an operator's bio)
- Posting: cross-posted from X.com threads, occasionally LinkedIn-native long-form

### 1.3 Studio B brand accounts

Studio B maintains brand accounts on X.com and LinkedIn that amplify Kevin's content. They don't have a separate voice — Kevin IS the brand at this stage.

**X.com @studiob (or equivalent):**
- Reposts/quote-tweets Kevin's high-signal posts
- Posts when a new Dispatch or DLI, by Studio B issue drops ("New issue of Dispatch, by Studio B →")
- Links to relevant product pages when Kevin's content touches Bolt or AcuOps territory

**LinkedIn Studio B page:**
- Same amplifier role
- Company updates when issues drop
- Follows the Billboard DNA in any visual assets used

### 1.4 What's not needed at launch

- Separate Capital, by Studio B social accounts — Kevin's personal presence covers the LP audience
- Separate Consulting, by Studio B accounts — same reason
- Instagram, TikTok, YouTube — not the audience

---

## Section 2: Newsletter Infrastructure

### 2.1 Dispatch, by Studio B (new publication)

| Item | Value |
|---|---|
| Publication name | Dispatch, by Studio B |
| Custom domain | `dispatch.b.studio` |
| DNS — CNAME | `dispatch` → `cname.beehiiv.com` |
| DNS — TXT | beehiiv ownership verification token (retrieved post-creation) |
| Pub ID | Captured via beehiiv API post-creation; used in consulting.b.studio embed |

**Subscribe page branding (beehiiv dashboard config):**
- Background: yolk `#F7D344`
- Button: ink `#0D0D0D`
- Headline font: Fraunces 900 (configured via custom CSS on beehiiv Max plan)
- Title: "Dispatch, by Studio B"
- Description: "War stories and field notes from Studio B. For operators who want to know what's actually happening inside the firms that are getting AI to work — not what the consultants are selling."

**HubSpot integration:**
- beehiiv native HubSpot integration enabled
- New subscriber → HubSpot contact created with `newsletter_source = "dispatch"`

### 2.2 DLI, by Studio B (existing publication — activate + brand)

Pub ID: `pub_e647b558-3a6e-4a29-87ff-c5fd2b566c35` (currently live at `brief.benchmarks.b.studio` — migrating to `DLI.b.studio`)

**DNS migration:**
- Add CNAME `DLI` → `cname.beehiiv.com` in b.studio GoDaddy zone
- Update beehiiv publication custom domain from `brief.benchmarks.b.studio` → `DLI.b.studio`
- Update beehiiv embed in `benchmarks-b-studio/index.html` from `brief.benchmarks.b.studio` → `DLI.b.studio`
- Old `brief.benchmarks.b.studio` CNAME can be removed after beehiiv confirms migration

**Subscribe page branding update:**
- Background: paper `#F4F1EB`
- Button: forest `#1A3328`
- Headline font: Fraunces 900
- Title: "DLI, by Studio B"
- Description: "We're building the open lower middle market lending index — and publishing the journey as we go. Quarterly index data, market event commentary, and field notes from the construction. Ground truth, not surveys."

**HubSpot integration:**
- beehiiv native HubSpot integration enabled
- New subscriber → HubSpot contact created with `newsletter_source = "dli"`

### 2.3 Build, by Studio B (new publication)

| Item | Value |
|---|---|
| Publication name | Build, by Studio B |
| Custom domain | `build.b.studio` |
| DNS — CNAME | `build` → `cname.beehiiv.com` |
| DNS — TXT | beehiiv ownership verification token (retrieved post-creation) |
| Pub ID | Captured via beehiiv API post-creation; used in build.b.studio embed |

**Subscribe page branding (beehiiv dashboard config):**
- Background: ink `#0D0D0D`
- Button: vermillion `#D94425`
- Headline font: JetBrains Mono (the build/code register; configured via custom CSS)
- Title: "Build, by Studio B"
- Description: "How we build AI-native software on real operations — and what breaks. Behind-the-scenes of Bolt, AcuOps, and everything else Studio B ships."

**Audience:** Technical founders, Acumatica ecosystem, operators who are also building. Content: product war stories, build-in-public on Bolt/AcuOps, lessons from shipping AI-native software in production environments.

**HubSpot integration:**
- beehiiv native HubSpot integration enabled
- New subscriber → HubSpot contact created with `newsletter_source = "build"`

### 2.4 Newsletter XP best practices (all publications)

Studio B holds beehiiv Max membership with access to Newsletter XP course. Apply best practices during setup:
- Welcome email sequence (3-issue onboarding)
- Subscribe page copy and headline optimization
- Confirmation email copy
- Referral program configuration (beehiiv native)

---

## Section 3: Website Layer

### 3.1 Platform

Both entity sites stay on **GitHub Pages**. No Webflow dependency. Billboard DNA carries across all Studio B properties: yolk `#F7D344`, ink `#0D0D0D`, vermillion `#D94425`, paper `#F4F1EB`, forest `#1A3328`, Fraunces 900, JetBrains Mono, Inter, diagonal caution stripes.

### 3.2 consulting.b.studio redesign

**Repo:** `studio-b-ai/consulting-b-studio`

**Current architecture:**
```
[caution stripe]
[hero] "We don't consult. We embed." → "Book a call" CTA
[3 work cards] ERP & Systems · AI & Automation · Growth & Capital
[HubSpot Calendly embed — full width]
[footer]
```

**Redesigned architecture:**
```
[caution stripe]
[hero] "We don't consult. We embed." → "Read Dispatch" CTA
[Dispatch subscribe section — primary]
  Headline: "Field notes from inside the operation."
  Body: 2-sentence description of what's in Dispatch
  beehiiv embed (Dispatch pub ID)
  Secondary line: "Also publishing DLI, by Studio B →" (link to DLI.b.studio/subscribe)
[3 work cards] ERP & Systems · AI & Automation · Growth & Capital
[engagement section]
  Headline: "Ready to go deeper?"
  Plain text Calendly link (no embed — one line, no iframe)
[footer]
```

The Calendly embed is removed. The call option is preserved as a plain text link — it's the upsell after the newsletter warms the relationship, not the hero action.

**Motion:** Intersection Observer scroll reveals on section entrance (~50 lines vanilla JS). Sections fade + translate-up on scroll into view.

### 3.3 capital.b.studio build

**Repo:** `studio-b-ai/capital-b-studio` (confirm existing repo or create new)

The LP customer journey is different from the consulting journey. LPs don't want to be sold to. They want to see analytical rigor, then reach out on their own terms. The DLI is the credibility signal — Studio B built the index because it didn't exist. That's the positioning.

**Page architecture:**
```
[hero]
  Headline: "We lend at the bottom of the market. And we measure it."
  Sub: "Capital, by Studio B is a private credit fund focused on LMM direct lending.
        We built the Studio B Direct Lending Index because the data didn't exist.
        Now we publish it."
  CTA: "Read DLI, by Studio B" → DLI.b.studio/subscribe

[DLI subscribe section — primary]
  Headline: "The only open index for LMM direct lending."
  Body: 2-sentence description of what DLI, by Studio B covers
  beehiiv embed (DLI pub ID)

[What we do]
  Fund thesis — 2-3 sentences, plain language
  The DLI as proof of process: "We built the benchmark because we needed it."
  3 conviction statements (equivalent of consulting.b.studio's work cards)

[LP inquiry section]
  Headline: "Interested in the fund?"
  One line: plain text email link or Calendly link (no form, no embed)
  Tone: low-pressure, for qualified LPs only

[footer]
```

**Branding:** Capital, by Studio B uses the paper `#F4F1EB` + forest `#1A3328` palette (same as DLI, by Studio B subscribe page). Fraunces 900 display, Inter body. Diagonal caution stripe dividers carry the Studio B DNA.

### 3.4 b.studio — hub redesign

`b.studio` is no longer just the firm homepage — it's the routing hub. Kevin's social presence points here. Visitors self-select their lane. The Billboard DNA stays; the architecture expands to present three service brand lanes clearly alongside the existing product panels.

**New section — three lanes:**
Position: between the hero and the product panels (Bolt + AcuOps), or as a dedicated section below them — whichever reads cleanest in the Billboard layout.

```
[Three lanes section]
  Headline: "Three ways to work with us."

  [Consulting, by Studio B]
    1-line description
    "Read Dispatch, by Studio B →" (consulting.b.studio)

  [Capital, by Studio B]
    1-line description
    "Read DLI, by Studio B →" (capital.b.studio)

  [Build, by Studio B]
    1-line description
    "Read Build, by Studio B →" (build.b.studio)
```

**Footer:** Add all three newsletter subscribe links.

### 3.5 build.b.studio

**Repo:** `studio-b-ai/build-b-studio` (new)

Audience is technical — operators who build, Acumatica ecosystem, founders. The register is direct and shows the work, not the pitch.

**Page architecture:**
```
[hero]
  Headline: "We ship on our own stack."
  Sub: "Bolt and AcuOps run Heritage Fabrics. We build in public."
  CTA: "Subscribe to Build, by Studio B" → beehiiv embed

[Build subscribe section]
  beehiiv embed (Build pub ID)

[What we're building]
  Bolt — one paragraph, plain language
  AcuOps — one paragraph, plain language
  Links to bolt.b.studio + acuops.com

[Recent build notes]
  3 recent Build, by Studio B issue excerpts + links

[footer]
```

**Branding:** ink `#0D0D0D` background with yolk `#F7D344` and vermillion `#D94425` accents. JetBrains Mono for display text alongside Fraunces. Heavier, more technical register than consulting or capital.

### 3.6 bibelhausen.com

**Repo:** `kbibelhausen/bibelhausen-com` (new, personal GitHub account — not studio-b-ai org)

Kevin's personal brand site. Covers all ventures, speaking, investment/angel activity. Points to the relevant Studio B entity depending on the visitor's context. Does not have its own newsletter.

**Visual identity:** Distinct from Studio B Billboard DNA. Clean, minimal, type-led. To be designed separately — the only constraint is it should not feel like a Studio B product page.

**Page architecture:**
```
[hero]
  Kevin Bibelhausen — one-line bio in his voice

[What I'm building]
  Studio B — 1 sentence + link to b.studio
  Capital, by Studio B — 1 sentence + link to capital.b.studio
  [Other ventures as they become public]

[Follow the work]
  Dispatch, by Studio B — subscribe link
  DLI, by Studio B — subscribe link

[Speaking + writing]
  Links to talks, published pieces (as they accumulate)

[Get in touch]
  Consulting inquiry → consulting.b.studio
  LP inquiry → capital.b.studio
  Everything else → email

[footer]
```

### 3.7 DLI.b.studio

The beehiiv-hosted subscribe/archive at `DLI.b.studio` is sufficient at launch. A custom-built standalone site is deferred — the capital.b.studio page handles the brand landing; `DLI.b.studio` handles the subscribe and archive functions that beehiiv provides natively.

---

## Section 4: CRM + Attribution

HubSpot is the attribution and relationship layer. It does not publish content. It tracks the full funnel for both entities from first newsletter subscribe through to conversion.

### 4.1 Contact properties

| Property | Type | Values | Set by |
|---|---|---|---|
| `newsletter_source` | Dropdown | `dispatch`, `dli`, `build`, `multiple` | beehiiv → HubSpot integration on subscribe |
| `subscriber_type` | Dropdown | `consulting_prospect`, `lp_prospect`, `operator`, `both` | Manual or enrichment |
| `utm_source` | Text | `dispatch`, `dli` | UTM tracking on click-through |
| `utm_campaign` | Text | `[issue-slug]` | UTM tracking on click-through |
| `calendly_booked_date` | Date | — | Calendly → HubSpot integration |
| `lp_inquiry_date` | Date | — | Manual or form submission |

### 4.2 Consulting pipeline

```
Subscriber (Dispatch, by Studio B)
  → Newsletter engagement (open rate, click-through to consulting.b.studio)
  → Calendly booked
  → Introductory call completed
  → Proposal sent
  → Engagement signed
```

### 4.3 Capital pipeline

```
Subscriber (DLI, by Studio B)
  → Newsletter engagement (open rate, click-through to capital.b.studio)
  → LP inquiry received
  → Introductory call completed
  → Materials sent (deck, LPA summary) — fund materials TBD; placeholder stage in pipeline until built
  → Committed LP
```

### 4.4 UTM attribution

All links in Dispatch, by Studio B and DLI, by Studio B use UTM parameters:
- `utm_source=dispatch` or `utm_source=dli`
- `utm_medium=newsletter`
- `utm_campaign=[issue-slug]`

HubSpot tracks the full funnel for both pipelines: subscribe → open → click → site → conversion action.

### 4.5 Reporting

One HubSpot dashboard per entity:
- **Consulting:** subscriber count, open rate, Calendly booked (last 30/90 days), pipeline value by newsletter source
- **Capital:** subscriber count, open rate, LP inquiry count, materials sent, committed LPs

---

## Section 5: Amplify Content Engine

### 5.1 Brand voice configs

Two brand voice entries in Amplify's per-brand config system, following the existing pattern for `studio-b`, `aesthetik`, `wvh`.

**`dispatch` voice config:**
```
Register: Consulting / operator-to-operator
Tone: Direct, no-BS, experienced. Kevin's voice. Like a dispatch from the field — not a newsletter.
Sentence style: Short declarative sentences. First person. Active voice.
What to include: A single incident or pattern from an active or recent engagement (anonymized).
  What happened, what we did, what the lesson is. Concrete details over abstractions.
What to avoid: Consulting-ese ("leverage," "synergies," "transformation journey").
  Bullet-point summaries. Anything that reads like a case study PDF. Inspirational sign-offs.
Sample opener: "We were three days into a systems audit when we found a $400K inventory
  discrepancy that nobody knew about. Here's what it told us."
Length target: 400–700 words per issue.
```

**`dli` voice config:**
```
Register: Benchmarks / build-in-public — but still Kevin's voice, not a data terminal
Tone: Curious, direct, practitioner-honest. Kevin happens to be building a lending index and he's
  telling you what he found. Not a quant publishing a white paper. Not a newsletter robot reciting
  spread compression. A person with opinions who also has the receipts.
Sentence style: Same short declarative sentences as Dispatch. First person. "We found that..."
  "Here's what the data is telling us." "This surprised me." Data is evidence, not the whole story.
What to include: The number or finding that matters, Kevin's read on what it means, one concrete
  implication for operators or LPs. Build-in-public: what we built, what broke, what we learned —
  told like a war story, not a commit log.
What to avoid: Academic register ("the data suggests a statistically significant..."). Pure data
  dumps with no interpretation. Any sentence that sounds like it belongs in a Cliffwater report.
  Quant jargon without a plain-English translation.
Sample opener: "The Q1 SBDLI spread compressed 85bps. That's not a data point — that's a signal
  that something changed in how LMM lenders are pricing risk. Here's what I think is happening."
Length target: 300–600 words for ad hoc issues. 600–900 words for quarterly index issues.
```

**`build` voice config:**
```
Register: Build-in-public / technical operator
Tone: Direct, specific, shows the work. Kevin built this and he's telling you exactly what happened.
  Not a product announcement. Not a dev blog. A war story from the build.
Sentence style: Same short declarative sentences. First person. "We shipped X. Here's what broke."
  "The thing nobody tells you about building on Acumatica is..." Technical details welcome —
  but always in service of the story, not the other way around.
What to include: What we built, why, what broke, what we learned. Specific version numbers,
  architectural decisions, the mistake that cost two days. Concrete over abstract.
What to avoid: Product announcement register ("We're excited to share..."). Changelog format.
  Marketing language. Anything that sounds like a press release or a SaaS blog post.
Sample opener: "We've been running Bolt on Heritage Fabrics' warehouse for six months.
  Here's the part of the stack that keeps breaking — and why we haven't fixed it yet."
Length target: 400–700 words per issue.
```

### 5.2 Amplify-Content — dialogue creation mode

Amplify has two content creation paths. The automated path (signal detected → draft generated) is reactive. Amplify-Content is the proactive path — Kevin-initiated, dialogue-driven, and the primary source of the best content.

**How it works:**

```
Kevin opens an Amplify-Content session
        ↓
System preloads context:
  - Recent sessions from Qdrant (what Kevin has actually been working on)
  - Recent Amplify signals (high-signal posts, market events, index updates)
  - Previous newsletter issues (what's already been covered)
        ↓
Amplify-Content asks probing questions one at a time:
  "You shipped the imports dashboard this week.
   What's the moment where you knew it was working?"
        ↓
Kevin responds (text or voice transcript)
        ↓
Amplify shapes the dialogue into:
  - Newsletter issue (prose, Kevin's voice)
  - X.com thread (sharpest lines from the dialogue)
  - LinkedIn post
  - Podcast episode transcript (the raw dialogue, ready for recording)
        ↓
Same editorial pipeline: Nael edits → Kevin approves via Slack → publishes
```

**The podcast format:**

The Amplify-Content dialogue doubles as podcast script. Kevin records himself answering the same questions he answered in the session — his real voice, natural setting. The questions are voiced by an ElevenLabs-generated producer voice. The dialogue (producer voice + Kevin voice) is edited together and published as an episode.

This format is transparent about the process — the producer is AI, Kevin isn't hiding it. That's on-brand for Studio B's build-in-public positioning and a live proof point for the firm's AI-leveraged approach.

**Production stack:**
- Amplify-Content generates the questions (text)
- ElevenLabs voices them as the producer character (audio); eventually an AI video avatar
- Kevin records his responses (phone or any simple audio setup — no studio required)
- Full recording: producer voice + Kevin voice captured together

**The editing room:**

The producer character is built in parallel with content shipping. Published episodes start as Kevin solo — the producer's audio is cut in the edit. When the producer voice (and eventually video) is polished enough to go public, the edit changes: keep the dialogue, ship Kevin + producer. No content backlog, no waiting. The character earns its place on-air.

```
Full recording: [producer question] + [Kevin answer] + [producer question] + [Kevin answer]...
        ↓ edit (solo phase)               ↓ edit (dialogue phase, when ready)
Kevin solo episode                   Kevin + producer dialogue episode
```

**The producer persona:**

Developed through internal content sessions. No name, no public identity until the character is coherent across multiple episodes. The ElevenLabs voice and (later) video avatar iterate privately. Ship when it's earned it.

**Build path:**

Phase 1 (now): Amplify-Content runs as a Claude Code skill or Claude.ai Project. Context loading from Qdrant + Amplify signals. Dialogue in the session. Text output only. Kevin records solo cuts from the transcript.

Phase 2 (Amplify M2 Console): ElevenLabs producer voice integrated. Full dialogue recording workflow. AI video avatar when character is ready to ship.

### 5.3 Social content outputs

Amplify drafts social content alongside newsletter issues — same signal, two outputs. This removes the gap between "newsletter drops" and "Kevin posts on X.com" — they're synchronized by default.

**For each newsletter issue, Amplify also drafts:**
- 1 X.com thread (3–5 posts): opening hook + 2–3 key points from the issue + subscribe CTA
- 1 LinkedIn post: slightly longer form, same substance, appropriate for professional register

**Social voice:**
- Shorter and punchier than the newsletter — X.com is the trailer, Dispatch is the film
- Same first-person, direct register as the newsletter voice configs
- No thread numbers ("1/", "2/") — just consecutive posts that flow
- CTA at the end: "Full dispatch at dispatch.b.studio" or "Full issue at DLI.b.studio"

**Editorial workflow for social content:**
- Delivered to Nael via Slack DM alongside the newsletter draft (same Amplify delivery event)
- Nael reviews with the same lens as newsletter editing
- Kevin approves social content in the same Slack approval message as the newsletter — one approval covers both

### 5.3 Signal sources

**Dispatch, by Studio B:**
| Signal type | Source | How Amplify detects |
|---|---|---|
| X.com engagement | Kevin's posts + replies + quote tweets | X.com API / Amplify signal worker |
| LinkedIn engagement | Kevin/Studio B content | Amplify L2 PhantomBuster → DIY engine (M1) |
| Field notes | Kevin's operator observations (manual input) | Direct input to Amplify queue |

X.com is the primary signal source. Until Amplify M4 (multi-platform expansion) ships, Dispatch signal detection runs on LinkedIn only. X.com is designed as primary in this spec so the voice config and signal map are correct from day one.

**DLI, by Studio B:**
| Signal type | Source | How Amplify detects |
|---|---|---|
| Quarterly index publication | SBDLI data pipeline (kbibelhausen/wasala repo) | Scheduled trigger on index publish |
| Market event | X.com / LinkedIn credit/LMM discussion | Signal scoring against SBDLI-relevant keywords |
| Build-in-public | kbibelhausen/wasala repo PRs merged | GitHub webhook → Amplify signal worker |

### 5.4 Editorial workflow

```
1. Signal detected (X.com, LinkedIn, repo event, quarterly trigger)
2. Amplify drafts:
   - Full newsletter issue in preset brand voice (complete prose — not bullet points)
   - X.com thread draft (3–5 posts)
   - LinkedIn post draft
3. All three delivered to Nael (newsletter draft in beehiiv, social drafts in Slack/Notion)
4. Nael reviews and edits for voice calibration, factual accuracy, length
5. Newsletter scheduled for publish in beehiiv
6. Kevin receives Slack notification:
   "📬 [Dispatch, by Studio B / DLI, by Studio B] issue ready for send — [headline] — approve?"
7. Kevin approves → issue sends + social posts go live
   Kevin requests edits → Nael revises → back to step 6
```

Voice tuning over time happens by updating the brand voice config in Amplify — not by correcting Nael issue-by-issue.

### 5.5 Cadence

**Dispatch, by Studio B:** Velocity-first. No committed schedule until 3 issues have run the full workflow. Target: biweekly once rhythm is set.

**DLI, by Studio B:** Quarterly baseline (anchored to SBDLI index publication). Plus ad hoc when a market event or notable build decision warrants it.

---

## Section 6: Delivery Sequence

### Pass 1 — Newsletter infrastructure

1. Create Dispatch, by Studio B beehiiv publication via API; capture pub ID
2. Create Build, by Studio B beehiiv publication via API; capture pub ID
3. Configure DNS for `dispatch.b.studio`, `DLI.b.studio`, and `build.b.studio` in GoDaddy
4. Brand all three subscribe pages via beehiiv dashboard
5. Enable HubSpot integrations on all three publications
6. Apply Newsletter XP best practices (welcome sequences, confirmation emails, referral program)
7. Update beehiiv embed in `benchmarks-b-studio/index.html` from `brief.benchmarks.b.studio` → `DLI.b.studio`

### Pass 2 — Social presence

1. Update Kevin's X.com bio + link-in-bio → `dispatch.b.studio/subscribe`
2. Update Kevin's LinkedIn featured section → both newsletter subscribe links
3. Update Kevin's LinkedIn About section in Kevin's voice
4. Confirm Studio B brand accounts exist on X.com + LinkedIn; update bios + links
5. Pin latest Dispatch issue (or "what I'm building" thread) on Kevin's X.com

### Pass 3 — Website layer

1. Rebuild `consulting-b-studio/index.html` — newsletter-first architecture (Section 3.2)
2. Wire Dispatch beehiiv embed (pub ID from Pass 1)
3. Add Intersection Observer scroll reveals; remove Calendly embed, replace with plain text link
4. Build `capital.b.studio` — newsletter-first, LP customer journey (Section 3.3)
5. Wire DLI, by Studio B beehiiv embed on capital.b.studio
6. Build `build.b.studio` — technical register, Bolt/AcuOps build-in-public (Section 3.5)
7. Wire Build, by Studio B beehiiv embed on build.b.studio
8. Redesign `b.studio` as three-lane hub — add service brand lanes section + footer links (Section 3.4)
9. Build `bibelhausen.com` — personal brand, own visual identity (Section 3.6); deploy from personal GitHub account
10. Deploy all via GitHub Pages

### Pass 4 — HubSpot CRM

1. Create custom contact properties (`newsletter_source`, `subscriber_type`, `lp_inquiry_date`)
2. Build Consulting pipeline (Section 4.2)
3. Build Capital pipeline (Section 4.3)
4. Connect Calendly → HubSpot integration
5. Build reporting dashboards (one per entity)

### Pass 5 — Amplify content engine

1. Add `dispatch` brand voice config to Amplify
2. Add `dli` brand voice config to Amplify
3. Map signal sources for both publications
4. Configure social content output (X.com thread + LinkedIn post per issue)
5. Wire Slack notification → Kevin on publish approval (newsletter + social in one message)
6. Configure Slack delivery of social drafts to Nael

### Pass 6 — Amplify-Content

1. Build Amplify-Content as a Claude Code skill (`/amplify-content`) — loads Qdrant session context + recent Amplify signals on open
2. Write the dialogue prompt: probing question set per publication (Dispatch register vs DLI register)
3. Wire output to standard draft pipeline (newsletter issue + social thread from dialogue transcript)
4. Run 3 internal content sessions; iterate on question quality and output shape
5. Integrate ElevenLabs producer voice — generate question audio from Amplify-Content text output
6. Establish recording workflow: Kevin records full dialogue (producer + Kevin); editor cuts producer for solo publish
7. Ship first episodes as Kevin solo while producer character iterates privately
8. Add podcast distribution: RSS feed, Apple Podcasts, Spotify (Dispatch audio companion)
9. When producer persona is coherent across 5+ episodes: publish dialogue format; optionally add AI video avatar

---

## Section 7: Revenue Model

This stack generates revenue beyond the primary conversion goals (engagements signed, LPs committed, product trials). Two parallel revenue streams are embedded in the architecture from day one.

### 7.1 HubSpot Solutions Partner

Studio B becomes a HubSpot Solutions Partner, earning 20% recurring commission on every HubSpot license under management.

**Immediate unlock:** Ästhetik is on HubSpot Enterprise at $5k/mo. As client #1, that's **$1k/mo ($12k/yr) in recurring revenue** from an existing relationship the moment Studio B is credentialed.

**Amplify multiplier:** Every Amplify client onboarded onto HubSpot through Studio B earns the same 20% recurring. At 10 clients averaging $800/mo (Professional), that's $1,600/mo additional.

**Prerequisites:**
- HubSpot Academy certifications (Marketing Software + Sales Software minimum) under Kevin's account
- Partner application submitted at https://www.hubspot.com/partners/solutions
- Confirm commission timing for existing Ästhetik contract (starts at renewal vs. immediately — verify with HubSpot partner team)
- Create Studio B HubSpot portal (separate from Ästhetik's existing portal)
- Connect Ästhetik as managed client #1

**Architecture:** Studio B HubSpot is the partner portal and hosts the three Studio B pipelines (consulting, capital, build). Ästhetik's existing portal remains intact as client #1 — all existing integrations (CS Order Entry, Bolt, Heritage Fabrics CRM) are unaffected.

**Parallel workstream — not a blocker to the marketing stack build.**

### 7.2 Certification Agent (AcuDev extension)

HubSpot Academy certifications run under Kevin's account via an agent — the same pattern as AcuDev's T-series capability (read course material, ingest knowledge base, pass multiple-choice exam). No human coursework required.

**Scope:** HubSpot Marketing Software, HubSpot Sales Software, and all other Academy certifications relevant to the Solutions Partner program. Generalizes to any platform certification (Salesforce, Google Ads, AWS, Klaviyo, etc.).

**Product implication:** When Amplify onboards a client onto the growth stack, certifications across all stack tools are part of the delivery. "Your team is certified" becomes an Amplify onboarding feature.

**Implementation:** AcuDev extension. Tracks under `project_acumatica-developer-agent.md` as next capability track. Not scoped in this implementation plan.

### 7.3 Tool affiliate stack

Every tool in the growth stack has an affiliate or referral program. Amplify's client onboarding flow routes new clients through Studio B's affiliate links, generating recurring passive revenue per client.

| Tool | Program | Rate |
|------|---------|------|
| HubSpot | Solutions Partner | 20% recurring (lifetime) |
| beehiiv | Affiliate | Recurring (verify rate at app.beehiiv.com/partners) |
| Transistor.fm | Affiliate | 30% recurring, 12 months |
| ElevenLabs | Affiliate | Verify at elevenlabs.io/affiliate |
| Typefully | Affiliate | Verify at typefully.com |

**Action:** Register affiliate accounts for each tool under Studio B before Amplify client onboarding begins. Bake affiliate links into the Amplify onboarding template (not the client-facing UI — internal onboarding checklist).

### 7.4 beehiiv native monetization

beehiiv Max includes two native revenue features that activate once subscriber volume warrants:

- **Boosts** — other newsletters pay to be recommended to your subscribers at sign-up. Financial/operator audience commands high CPMs. Enable in beehiiv settings; no editorial work required.
- **Ad Network** — sponsored placements in issues. DLI especially: credit data vendors, fintech, law firms. Enable once DLI reaches ~500 subscribers.

**Paid tier (DLI):** beehiiv supports premium paid subscriptions. DLI is the strongest candidate for a paid tier — credit professionals pay for data and proprietary index access. Revisit at 500 subscribers.

---

## Open Questions

- **capital.b.studio repo:** New `studio-b-ai/capital-b-studio` repo, or does an existing repo serve this? Confirm before Pass 3.
- **bibelhausen.com visual identity:** To be designed separately. Constraint: distinct from Studio B Billboard DNA — clean, minimal, personal. Confirm palette + typography before Pass 3 step 9.
- **bibelhausen.com GitHub account:** Confirm whether repo lives under `kbibelhausen` personal account or `studio-b-ai` org.
- **Kevin's X.com handle:** Confirm the primary account handle for profile updates in Pass 2.
- **Build, by Studio B signal sources:** What signals trigger Build issues? GitHub PRs merged (bolt-wms, acuops-pipeline)? Manual field notes only? Confirm for Amplify voice config in Pass 5.
- **HubSpot Calendly integration:** Confirm Calendly → HubSpot native integration is already set up, or needs to be wired.
- **Referral program:** Skip referrals at launch for all three newsletters, or configure for Dispatch only?
- **Amplify M4 / X.com signal worker:** Confirm whether M4 timeline affects Pass 5 sequencing.
- **Build pipeline conversion goal:** Build, by Studio B audience converts to what? Product trial (Bolt/AcuOps)? Consulting engagement? Amplify pipeline referral? Confirm for HubSpot pipeline setup.
