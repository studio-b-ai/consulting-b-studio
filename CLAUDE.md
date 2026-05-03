# Studio B Marketing Stack

This repo is the **hub for the Studio B marketing stack build** — the Amplify reference client implementation.

Studio B is dogfooding Amplify. Every design decision is also a product decision. Build it right for Studio B; resell the pattern as Amplify.

## What this project covers

Five repos ship under this umbrella:

| Repo | Domain | Entity |
|------|--------|--------|
| `studio-b-ai/consulting-b-studio` | `consulting.b.studio` | Consulting, by Studio B |
| `studio-b-ai/capital-b-studio` | `capital.b.studio` | Capital, by Studio B |
| `studio-b-ai/build-b-studio` | `build.b.studio` | Build, by Studio B |
| `studio-b-ai/b-studio-website` | `b.studio` | Studio B hub |
| `kbibelhausen/bibelhausen-com` | `bibelhausen.com` | Kevin Bibelhausen (personal) |

Spec: `docs/superpowers/specs/2026-05-02-dispatch-sbdli-amplify-design.md`

## MCP tools available

- **beehiiv** — read/write publications, posts, subscribers. Pub IDs in spec §2.
- **studiob** — studiob-api gateway (Acumatica, Slack, internal tooling)
- **HubSpot** — already wired at the Claude account level. Three pipelines: consulting, capital, build.
- **ElevenLabs** — configured in `.mcp.json`; requires `ELEVENLABS_API_KEY` (see onboarding checklist below)
- **Typefully** — configured in `.mcp.json`; requires `TYPEFULLY_API_KEY` (see onboarding checklist below)

## Credentials needed (onboarding checklist)

Before implementing anything that requires these tools:

### ✅ Ready
- `gh` CLI — authenticated as `kbibelhausen`
- beehiiv MCP — live with API key
- HubSpot MCP — connected at Claude account level
- studiob MCP — live with bearer token
- 1Password — authenticated

### ⚠️ Needs API key
- **ElevenLabs** — get from https://elevenlabs.io/app/settings/api-keys → update `ELEVENLABS_API_KEY` in `.mcp.json`
- **Typefully** — get from https://app.typefully.com/settings/api → update `TYPEFULLY_API_KEY` in `.mcp.json`
- **Transistor.fm** — REST API only (no MCP). Key from https://dashboard.transistor.fm/account → store in 1Password as "Transistor.fm API Key"
- **Railway CLI** — `railway login` (session expired)

### Manual steps (Claude cannot do these)
- beehiiv Max upgrade — https://app.beehiiv.com/settings/billing (required for custom domains + 3-pub plan)
- Create 3 beehiiv publications: Dispatch, DLI, Build (each needs custom domain configured)
- HubSpot: create consulting / capital / build pipelines and forms
- Transistor.fm: create podcast show "Build, by Studio B"
- Descript: account setup for audio editing

## Repo conventions

All five sites are **GitHub Pages static HTML** using the Studio B Billboard design system.

Design system canonical: `/Users/kevin/.claude/projects/-Users-kevin-dev/memory/context/b-studio-design-system.md`

### Billboard palette (all sites except bibelhausen.com)
- Yolk `#F7D344` — hero backgrounds
- Ink `#0D0D0D` — body text, labels
- Vermillion `#D94425` — accents, CTAs
- Forest `#1A3328` — secondary
- Paper `#F4F1EB` — light sections

### Build, by Studio B register
Ink `#0D0D0D` background, vermillion + yolk accents, JetBrains Mono display — technical/build-in-public

### bibelhausen.com
Distinct visual identity from Billboard. Design TBD — do NOT apply Billboard DNA here.

## beehiiv known pub IDs

| Publication | Pub ID |
|-------------|--------|
| SBDLI Brief (existing) | `pub_e647b558-3a6e-4a29-87ff-c5fd2b566c35` |
| Dispatch, by Studio B | TBD — create in beehiiv |
| DLI, by Studio B | TBD — create in beehiiv |
| Build, by Studio B | TBD — create in beehiiv |

## Naming convention

Always: `[Name], by Studio B` with a comma before "by".

- Dispatch, by Studio B ✓
- DLI, by Studio B ✓
- Build, by Studio B ✓
- Consulting, by Studio B ✓
- Capital, by Studio B ✓

Never: "Studio B Dispatch", "Dispatch by Studio B" (no comma)

## Implementation plan files

After the environment is ready, implementation plans live at:
- `docs/superpowers/plans/YYYY-MM-DD-websites.md`
- `docs/superpowers/plans/YYYY-MM-DD-newsletter-infra.md`
- `docs/superpowers/plans/YYYY-MM-DD-hubspot-crm.md`
- `docs/superpowers/plans/YYYY-MM-DD-amplify-content.md`

## Amplify signal sources

- Dispatch: Kevin's X.com posts (via Amplify M4 signal detector)
- DLI: wasala repo PRs + quarterly SBDLI index publish
- Build: X.com + GitHub activity (acuops-pipeline, bolt-wms, amplify repos)
