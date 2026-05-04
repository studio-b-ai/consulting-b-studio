# Studio b. — Architecture & Sales Motions

**Date:** 2026-05-04
**Status:** Locked through 2026-05-04 architecture sequence. Premium tier work gated behind wasala M9.
**Owner:** Kevin Bibelhausen
**Supersedes:** project-level decisions in `memory/project_studio-b-positioning.md` Q2 (revised 2026-05-04 to reflect Capital publicly marketed). Carries through `wasala/docs/master-plan.md` freemium-launch framing.

---

## TL;DR

Studio b. operates as an umbrella firm with three commercial branches that share a unified naming convention but run independent funnels:

1. **Operator side (Markdown)** — the operator newsletter at `markdown.b.studio`. Funnel into consulting services and product trials.
2. **Financial-services side (LMMI)** — Studio b. LMMI is the Lower Middle Market Direct Lending Index. Two parallel tracks downstream of the free Tier 0/1 layer:
   - **Premium track** (paid sub on `lmmi.b.studio`, M9-gated) — non-LP institutional readers
   - **LP track** (`invest.b.studio` → 506(c) self-attestation → backtester) — accredited prospective LPs of Studio b. LMM Bridge Fund I
3. **Products** — Bolt, AcuOps, Amplify. Each independent brand owns its own funnel; b.studio + Markdown act as discovery surfaces only.

**Locked anti-pattern:** do NOT paywall the LMMI backtester. It's the LP conversion artifact. Premium track is for an *entirely different audience* (non-LPs). Paywalling adds friction to the LP funnel and defeats the purpose.

---

## 1. The architecture

```
                          ┌──────────────────────┐
                          │      Studio b.       │  ← operating firm umbrella
                          │      b.studio        │     (paper / Billboard register)
                          └──────────┬───────────┘
                                     │
        ┌────────────────────────────┼────────────────────────────────┐
        │                            │                                │
   ┌────▼──────┐              ┌──────▼──────┐                  ┌──────▼─────────┐
   │ Operator  │              │  Financial  │                  │   Products     │
   │   side    │              │   services  │                  │  (independent  │
   │           │              │             │                  │     brands)    │
   │  Markdown │              │  LMMI brand │                  │                │
   │ markdown  │              │             │                  │  bolt.b.studio │
   │ .b.studio │              │ ┌─────────┐ │                  │  acuops.com    │
   │           │              │ │  Tier 0 │ │                  │ amplify.b.studio
   │ Audience: │              │ │  free   │ │                  │                │
   │  ops +    │              │ │  ticker │ │                  │ Each handles   │
   │  tech     │              │ ├─────────┤ │                  │ its own        │
   │  founders │              │ │  Tier 1 │ │                  │ marketing,     │
   │           │              │ │  email  │ │                  │ pricing,       │
   │ HubSpot:  │              │ │  sub    │ │                  │ funnels.       │
   │  Studio B │              │ └────┬────┘ │                  │                │
   │  Markdown │              │      │      │                  │ Markdown is    │
   │  pipeline │              │      ├──────┴───────┐          │ their share-of-│
   │           │              │      │              │          │ voice surface  │
   └───────────┘              │ ┌────▼─────┐  ┌─────▼──────┐  │ ("What we ship│
                              │ │ Premium  │  │ LP track   │  │  on Markdown") │
                              │ │ track    │  │            │  └────────────────┘
                              │ │          │  │ invest.b.  │
                              │ │ paid sub │  │ studio     │
                              │ │ on lmmi  │  │            │
                              │ │ .b.studio│  │ 506(c)     │
                              │ │          │  │ self-      │
                              │ │ M9-gated │  │ attestation│
                              │ │ ~$5–15K/ │  │            │
                              │ │ yr       │  │ Backtester │
                              │ │          │  │ access     │
                              │ │ For non- │  │ (free)     │
                              │ │ LP allo- │  │            │
                              │ │ cators / │  │ For        │
                              │ │ researchers│ accredited │
                              │ │ / press  │  │ LPs        │
                              │ └──────────┘  └─────┬──────┘
                              │                     │
                              │              ┌──────▼──────────────┐
                              │              │ Studio b. LMM       │
                              │              │ Bridge Fund I       │
                              │              │ (legal: Wasala BVI) │
                              │              │                     │
                              │              │ HubSpot: LMMI       │
                              │              │ pipeline            │
                              │              └─────────────────────┘
                              └─────────────────────────────────────┘

                            301 redirects (live):
                              consulting.b.studio → markdown.b.studio
                              capital.b.studio   → lmmi.b.studio
                              build.b.studio     → markdown.b.studio
                              dispatch.b.studio  → markdown.b.studio
                              dli.b.studio       → lmmi.b.studio
```

### Surface inventory (live)

| Surface | Domain | Audience | Conversion goal | Status |
|---|---|---|---|---|
| Hub | `b.studio` | All | Route by intent | ✅ live |
| Markdown | `markdown.b.studio` | Operators + technical founders | Newsletter sub → consulting / product trial | ✅ live |
| LMMI public | `lmmi.b.studio` | Anyone | Newsletter sub | ✅ live |
| Capital LP entry | `invest.b.studio` | Accredited LPs | Backtester access → fund commitment | ✅ live |
| Bolt | `bolt.b.studio` | Continuous-goods distributors | WMS subscription | ✅ live (independent) |
| AcuOps | `acuops.com` | Acumatica VARs | $300/mo/instance | ✅ live (independent) |
| Amplify | `amplify.b.studio` | Founders + marketing teams | Subscription | ✅ live (independent) |
| Premium LMMI | `lmmi.b.studio` paywall | Non-LP institutional readers | $5–15K/yr sub | 🔵 M9-gated |

### Brand naming (canonical 2026-05-04)

- **Studio b.** — operating firm umbrella
- **Markdown, by Studio b.** — operator newsletter
- **Studio b. LMMI** (Lower Middle Market Direct Lending Index) — financial-services flagship publication. Pronounced "L-M-M-I." **Always prefix with "Studio b." in public copy** — disambiguates from Cliffwater's CDLI which dominates "DLI" as a mnemonic in institutional credit.
- **Studio b. LMM Bridge Fund I** — fund vehicle trade name
- **Wasala** — fund BVI legal entity (back-end name; no public marketing)
- **Capital, by Studio b.** — retired as public brand 2026-05-04
- **Future fund names** — Studio b. LMM Bridge Fund II, Studio b. LMM Credit Fund II, Studio b. LMM Index Fund (the Cliffwater-pattern interval RIC)

### Two-track LMMI conversion (locked 2026-05-04)

```
   Tier 0 (free public)     ←  lmmi.b.studio quarterly headline + ticker
        +                       (citable, Cliffwater-equivalent free layer)
   Tier 1 (free email)      ←  newsletter subscription + market commentary
                                 ↓
                                splits by audience
            ┌────────────────────┴────────────────────┐
            │                                         │
    Premium track                              LP track
    (paid sub, M9)                             (Rule 506(c) gate, shipped)
    ───────────────                            ─────────────────────────
    Methodology PDF                            BACKTESTER ACCESS
    Constituent universe                       Custom backtest scenarios
    Historical CSV (10y+)                      Methodology consultation
    Weekly commentary                          Direct line to Kevin
    DD-grade brief
    ───────────────                            ─────────────────────────
    audience: allocators,                      audience: prospective LPs
              researchers,                                (accredited only)
              journalists,
              competitors                      conversion: → fund commitment
    conversion: → subscription                 cost: free (price friction
                  revenue                            on LP funnel kills it)
    cost: $5–15K/yr
```

---

## 2. Sales motions per audience

### Motion 1 — Operator (services + product trials)

**Audience:** Operators, founders, GMs running real businesses (~$10M–$500M rev). The Heritage Fabrics / Weathervane Hill / Roth-class buyer.

**Funnel:**
```
Discovery: Markdown subscriber (email opt-in via embed)
   → Engagement: clicks through to markdown.b.studio
       → Pull: "Work with us" inquiry → hello@b.studio
       OR Pull: Product trial request → bolt.b.studio / acuops.com / amplify.b.studio
           → Each product owns its own pipeline from there
```

**HubSpot pipeline:** **Studio B Markdown** (renamed from Build).
Stages: Markdown Subscriber → Engaged (clicked through to markdown.b.studio) → Product Trial or Meeting Requested → Converted.

**Conversion paths:**
1. **Consulting work** — fractional CTO retainers ($15–30K/mo), AI ops transformation ($50–150K), Acumatica strategy ($25–50K). Per `memory/project_studio-b-positioning.md` services lineup. Engagement floor $250K/yr per locked tuning proposal C.
2. **Product trial** — handed off to product brand. Bolt $9,600–$21,600/yr per Acumatica instance; AcuOps $3,600/yr flat per Acumatica instance; Amplify TBD.

**Gap:** no dedicated "Work with us" surface. `hello@b.studio` is the catch-all. If services becomes a real revenue line, a `markdown.b.studio/work` page or equivalent is the natural addition.

### Motion 2 — LP / Accredited Investor (Studio b. LMM Bridge Fund I)

**Audience:** Accredited investors, family offices, qualified purchasers. First-close target: 1 GP money partner ($2M convertible) + 1 anchor LP ($10–20M). Ongoing LPs $250K–$2M+.

**Funnel:**
```
Discovery: free LMMI ticker on lmmi.b.studio OR b.studio hub OR organic search
   → Engagement: subscribes to LMMI newsletter (Tier 1, free)
      → Indication: clicks through to invest.b.studio
         → Self-attestation: 506(c) accredited checkbox
            → Backtester access request (HubSpot form, LMMI · Backtester Access)
               → Routed into LMMI pipeline at "LP Inquiry Received" stage
                  → Intro call with Kevin
                     → Backtester access shared (LP runs scenarios)
                        → Tear sheet v5 + methodology shared post-attestation
                           → Third-party accreditation verification
                              (CPA / attorney / verifier)
                              → Fund I subscription docs
                                 → Committed LP
```

**HubSpot pipeline:** **LMMI** (renamed from Studio B Capital).
Stages: LMMI Subscriber → Engaged (clicked through to dli.b.studio — needs URL update to lmmi.b.studio) → LP Entry (invest.b.studio) → LP Inquiry Received → Introductory Call Completed → Materials Sent → Committed LP.

**Conversion artifact:** The **backtester** (NOT a deck).

> Per Kevin's framing 2026-05-04: "we don't provide a deck as much as we provide them a model against our index that they back test against. We'll use the model to sharpen our thesis."

The backtester run is also a feedback loop — LP scenarios sharpen Studio b.'s underwriting thesis. Free to verified accredited investors; gated by Rule 506(c) self-attestation, NOT by payment.

**Forbidden language** (per v4 canonical, see `memory/reference_capital-positioning.md`):
- No "private credit" — use "non-bank lender"
- No QAHWA reference
- No "$1B AUM as forecast" (it's a stated goal, phrase accordingly)
- No public tear-sheet links
- No MENA sovereign names

**Compliance gates:**
- 506(c) self-attestation = soft gate at backtester request (filters tire-kickers, low-friction)
- Third-party accredited verification = hard gate before subscription (Rule 506(c) requires: CPA letter / attorney letter / Parallel Markets verification or equivalent)
- Tear sheet v5 / LPA / PPM / subscription docs = post-verification artifacts only

### Motion 3 — Premium institutional reader (LMMI subscription, M9+)

**Audience:** Allocators, family-office DD teams, journalists/researchers, banks doing market sizing, competing fund managers doing competitive intel. Anyone who wants institutional LMM data without the LP relationship. Cliffwater CDLI buyers paying ~$25K/yr — same buyer pool, undercut by LMM specificity (CDLI tracks the BDC universe which is structurally upper-middle).

**Funnel:**
```
Discovery: free LMMI ticker / Tier 1 newsletter
   → Engagement: clicks "Subscribe for full methodology" on lmmi.b.studio
      → Stripe / beehiiv Boost paid sub ($5–15K/yr)
         → Recurring revenue
```

**Conversion goal:** subscription revenue, not fund commitment. Discrete revenue line.

**What's in it:**
- Full IOSCO-compliant methodology PDF
- Constituent universe disclosure
- Downloadable historical CSV (10y+)
- Weekly market commentary
- CDLI peer comparison
- Quarterly DD-grade brief

**What's NOT in it:** the backtester. Backtester is LP-track only.

**Status:** 🔵 M9-gated per `wasala/docs/master-plan.md`. Don't ship the paywall until M9 (~Q3 2026 calendar, depending on roadmap pace).

### Motion 4 — Product customers (Bolt / AcuOps / Amplify)

**Audience:** Each product has its own ICP — Bolt (continuous-goods distributors), AcuOps (Acumatica VARs), Amplify (founders + marketing teams).

**Funnel:** each product brand owns its own. Studio b. hub + Markdown act as **discovery surfaces** ("What Studio b. ships" section), but conversion happens on the product domains. Out of scope for the firm-level architecture.

---

## 3. Money flows

| Flow | Audience | Surface | Annual revenue per unit |
|---|---|---|---|
| Services retainer | Operator | hello@b.studio funnel | $180–360K/yr (fractional CTO) |
| Services fixed | Operator | hello@b.studio funnel | $50–250K per engagement |
| Bolt subscription | Distributor | bolt.b.studio | $9,600–$21,600/yr per instance |
| AcuOps subscription | VAR | acuops.com | $3,600/yr per Acumatica instance |
| Amplify subscription | Founder/team | amplify.b.studio | TBD (varies) |
| LMMI premium sub | Allocator/researcher | lmmi.b.studio paywall (M9+) | $5–15K/yr |
| Fund I LP commitment | Accredited LP | invest.b.studio → fund | 2% mgmt + 20% carry over 10% pref |

**Flywheel:** Tier 0 LMMI is the audience flywheel (free, citable, builds Kevin's thought leadership). Tier 1 captures email. From there, two financial-services revenue lines: premium subscription + fund commitment. Plus operator-side services + product subscriptions running in parallel.

---

## 4. HubSpot pipeline state

| Pipeline | ID | Stages |
|---|---|---|
| **Studio B Markdown** (renamed from Build) | `895657270` | Markdown Subscriber (5%) → Engaged (clicked through to markdown.b.studio) (15%) → Product Trial or Meeting Requested (40%) → Converted (Won 100%) |
| **LMMI** (renamed from Studio B Capital) | `895657269` | LMMI Subscriber (5%) → Engaged (clicked through to dli.b.studio — needs URL update) (15%) → LP Inquiry Received (30%) → LP Entry (invest.b.studio) (20%) [needs reorder] → Introductory Call Completed (50%) → Materials Sent (65%) → Committed LP (Won 100%) |
| Studio B Consulting | (deleted 2026-05-04) | — |

**Form:** **LMMI · Backtester Access** (form ID `8029f7ca-b512-4922-b56d-34d2dff1bfbd`) — wired into invest.b.studio. Renamed and republished 2026-05-04. Currently has First Name / Last Name / Email only; needs Company / Job Title / Investor type / Ticket interest / 506(c) consent fields added manually in HubSpot UI.

---

## 5. Open gaps / queued work

1. **HubSpot LP Entry stage position** — currently at position 4 in LMMI pipeline; needs manual drag to position 3 (between Engaged and LP Inquiry Received). 3-second fix in HubSpot UI.
2. **HubSpot stage URL label** — `Engaged (clicked through to dli.b.studio)` still references the old URL (which now redirects). Cosmetic update to `Engaged (clicked through to lmmi.b.studio)`.
3. **Form fields on LMMI · Backtester Access** — currently First Name / Last Name / Email only. Should add: Company name, Job title, Investor type (dropdown: Individual accredited / Qualified purchaser / Institution / Family office), Ticket interest (dropdown: $25K-$100K / $100K-$500K / $500K-$2M / $2M+), Notes / Why interested, 506(c) consent checkbox. Custom dropdowns + consent checkbox need to be created as HubSpot properties first.
4. **HubSpot workflow** — wire form submission to create deal in LMMI pipeline at "LP Inquiry Received" stage. Currently no automation.
5. **Services intake surface** — `hello@b.studio` is a generic catch-all. If services becomes a real revenue line, consider `markdown.b.studio/work` with structured intake form routing into the Markdown pipeline.
6. **Premium tier build (M9-gated)** — Stripe / beehiiv Boost paywall on `lmmi.b.studio` with methodology PDF + constituent universe + historical CSV + weekly commentary + CDLI peer comparison + DD brief. Don't ship before wasala M9.
7. **Tear sheet v5 + 8 governance PDFs rename** — queued behind M7 in v1.2 redline window. Folds in the rename from "Capital, by Studio b." → "Studio b. LMM Bridge Fund I" (trade name). Wasala stays as legal BVI entity.
8. **Services lineup public surface** — per `memory/project_studio-b-positioning.md`, six service offerings exist (CTO retainer / AI transformation / Acumatica strategy / MVP / diligence / managed services). With consulting.b.studio retired, the services lineup needs a public home if services revenue is a priority. Decision pending.
9. **wasala roadmap M9 calendar date** — when is freemium launch realistically? Affects when premium tier ships. Pull from `wasala/docs/master-plan.md` milestone schedule when needed.

---

## Memory canonical references

- `memory/context/b-studio-design-system.md` — newsletter family + brand architecture + two-track conversion (locked 2026-05-04)
- `memory/project_studio-b-positioning.md` — Q1–Q5 (Q2 revised 2026-05-04)
- `memory/reference_capital-positioning.md` — v4 canonical for fund structure + voice + forbidden language; rename banner at top
- `memory/project_capital-fund-formation.md` — Wasala BVI / DE feeder structure, Fund II+ roadmap
- `wasala/docs/master-plan.md` — DLI/LMMI publishing roadmap, M9 freemium gate
- `wasala/docs/plans/2026-04-28-m3.5-dli-implementation-plan.md` — index machinery implementation

## Live infrastructure references

- Hub: https://b.studio
- Markdown: https://markdown.b.studio
- LMMI: https://lmmi.b.studio (dli.b.studio redirects)
- Invest: https://invest.b.studio
- Bolt / AcuOps / Amplify: independent product domains
- HubSpot portal: 49070660
- HubSpot pipelines: LMMI (`895657269`), Studio B Markdown (`895657270`)
- HubSpot form: LMMI · Backtester Access (`8029f7ca-b512-4922-b56d-34d2dff1bfbd`)
