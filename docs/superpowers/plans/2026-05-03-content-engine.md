# Content Engine Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Wire Amplify for newsletter output — three brand voice configs (Dispatch, DLI, Markdown), a beehiiv draft adapter that pushes generated content as beehiiv drafts, GitHub and LinkedIn signal sources, Typefully for X.com scheduling, and the Amplify-Content Claude Code skill for Kevin-initiated dialogue sessions.

**Architecture:** Amplify's existing `apps/engine/src/config/brands/` YAML system extends naturally to newsletter brands. The generation pipeline already uses Claude to produce content; adding beehiiv as an output channel requires a new `BeehiivDraftAdapter` in `apps/workers/`. Signal sources for newsletter triggers (GitHub PR merges, quarterly SBDLI trigger) wire into the existing engagement-worker ingest path. Amplify-Content is a Claude Code skill (`/amplify-content`) that loads Qdrant session context and runs interactive dialogue sessions.

**Tech Stack:** TypeScript (Amplify), YAML brand configs, beehiiv API, Typefully MCP (already in `.mcp.json`), GitHub webhook (existing webhook-router), BullMQ workers, Claude (sonnet-4-6).

**Repos:**
- `studio-b-ai/amplify` — brand configs, beehiiv adapter, signal source wiring
- `studio-b-ai/webhook-router` — GitHub signal ingest routes
- `~/.claude/skills/amplify-content/` — Amplify-Content Claude Code skill

**Dependency:** Newsletter infra plan must be complete — beehiiv pub IDs from `docs/pub-ids.md` are required in Task 2 (beehiiv adapter) and Tasks 3-4 (signal source configs).

---

## File Structure

**Create in `studio-b-ai/amplify`:**
- `apps/engine/src/config/brands/dispatch.yaml` — Dispatch brand voice config
- `apps/engine/src/config/brands/dli.yaml` — DLI brand voice config
- `apps/engine/src/config/brands/markdown.yaml` — Markdown brand voice config
- `apps/workers/src/adapters/beehiiv-draft.ts` — beehiiv draft writer
- `apps/workers/src/adapters/beehiiv-draft.test.ts` — unit tests
- Modify: `apps/workers/src/content-worker.ts` — add beehiiv publish step + newsletter generation mode

**Create in `~/.claude/skills/amplify-content/`:**
- `skill.md` — Amplify-Content skill definition

---

## Task 1: Dispatch brand voice config

**Files:**
- Create: `studio-b-ai/amplify/apps/engine/src/config/brands/dispatch.yaml`

**Work in:** `/Users/kevin/dev/amplify/`

- [ ] **Step 1: Write failing test**

Create `apps/engine/src/config/brands/__tests__/dispatch-config.test.ts`:

```typescript
import { loadBrandConfig } from "../../lib/brand-loader.js";
import path from "path";
import { fileURLToPath } from "url";

const __dirname = path.dirname(fileURLToPath(import.meta.url));

test("dispatch config loads without error", () => {
  const configPath = path.join(__dirname, "../dispatch.yaml");
  expect(() => loadBrandConfig(configPath)).not.toThrow();
});

test("dispatch config has required fields", () => {
  const configPath = path.join(__dirname, "../dispatch.yaml");
  const config = loadBrandConfig(configPath);
  expect(config.brand.name).toBe("Dispatch, by Studio B");
  expect(config.brand.voice).toBeTruthy();
  expect(config.brand.audience).toBeTruthy();
  expect(config.channels).toBeDefined();
});
```

Run: `cd /Users/kevin/dev/amplify && npm test --workspace apps/engine -- --testPathPattern dispatch-config`  
Expected: FAIL — dispatch.yaml not found.

- [ ] **Step 2: Create dispatch.yaml**

Create `apps/engine/src/config/brands/dispatch.yaml`:

```yaml
brand:
  name: "Dispatch, by Studio B"

  voice: |
    Kevin's voice. Operator/consultant register. Like a dispatch from the field.
    
    Register: Consulting / operator-to-operator
    Tone: Direct, no-BS, experienced. Written as if sent between the incident and
      the post-mortem — not polished, not advisory. Kevin talking to an operator.
    Sentence style: Short declarative sentences. First person. Active voice.
      "We were three days into the audit." "Here's what it told us."
    What to include: A single incident or pattern from an active or recent
      engagement (anonymized). What happened, what was done, what the lesson is.
      Concrete details over abstractions — the specific system, the specific number,
      the specific person's reaction.
    What to avoid: Consulting-ese ("leverage", "synergies", "transformation journey").
      Bullet-point summaries. Case study PDF format. Inspirational sign-offs.
      Anything that sounds like advice from someone who hasn't been in the room.
    Sample opener: |
      We were three days into a systems audit when we found a $400K inventory
      discrepancy that nobody knew about. Here's what it told us.
    Length target: 400-700 words per issue.
    Output format: Full newsletter issue as prose. Not bullets. Not headers.
      A story with a lesson.

  voice_reference: "memory/feedback_studio-b-voice.md"

  audience: |
    Operators, PE-backed company principals, fractional CTOs, and LMM executives
    who want to know what's actually happening inside the firms that are getting AI
    to work — not what the consultants are selling. They're running companies, not
    reading theory. They want the field report.

  forbidden:
    - "seamless"
    - "intuitive"
    - "best-in-class"
    - "enterprise-grade"
    - "comprehensive"
    - "cutting-edge"
    - "revolutionary"
    - "game-changing"
    - "synergy"
    - "leverage"
    - "paradigm"
    - "disrupt"
    - "end user"
    - "end-user"
    - "transformation journey"
    - "AI-powered"
    - "next-gen"
    - "world-class"
    - "we believe"
    - "may help you"
    - "could potentially"

content:
  library_source: ""
  types:
    - newsletter_issue
    - social_thread_x
    - social_post_linkedin

channels:
  beehiiv:
    publication_id: "DISPATCH_PUB_ID"  # replace with actual pub ID from docs/pub-ids.md
    adapter: beehiiv_draft
    output_type: newsletter_issue
  x_thread:
    adapter: typefully
    output_type: social_thread_x
    posts_in_thread: 4  # opening hook + 2 key points + subscribe CTA
  linkedin:
    adapter: hubspot
    output_type: social_post_linkedin
    format: text_post

signal_sources:
  linkedin_engagement:
    description: "Kevin's LinkedIn posts, engagements, comment threads — primary trigger for Dispatch issues"
    adapter: linkedin_signal
    brand_filter: "studio-b"
    min_signal_score: 65
  manual_field_note:
    description: "Kevin submits a field note directly to Amplify signal queue via Slack command"
    adapter: slack_intake
    channel: "#raw-content-studiob"
```

- [ ] **Step 3: Run test to verify it passes**

```bash
cd /Users/kevin/dev/amplify
npm test --workspace apps/engine -- --testPathPattern dispatch-config
```

Expected: PASS.

- [ ] **Step 4: Replace DISPATCH_PUB_ID placeholder**

Read `docs/pub-ids.md` from the consulting-b-studio repo. Replace `DISPATCH_PUB_ID` in dispatch.yaml with the actual Dispatch publication ID.

- [ ] **Step 5: Ship with /ship**

Invoke `/ship` with message `"feat(brands): add Dispatch, by Studio B brand voice config"`.

---

## Task 2: DLI and Markdown brand voice configs

**Files:**
- Create: `studio-b-ai/amplify/apps/engine/src/config/brands/dli.yaml`
- Create: `studio-b-ai/amplify/apps/engine/src/config/brands/markdown.yaml`

- [ ] **Step 1: Write failing tests**

Add to `apps/engine/src/config/brands/__tests__/dispatch-config.test.ts` (same test file, extend it):

```typescript
test("dli config loads without error", () => {
  const configPath = path.join(__dirname, "../dli.yaml");
  const config = loadBrandConfig(configPath);
  expect(config.brand.name).toBe("DLI, by Studio B");
});

test("markdown config loads without error", () => {
  const configPath = path.join(__dirname, "../markdown.yaml");
  const config = loadBrandConfig(configPath);
  expect(config.brand.name).toBe("Markdown, by Studio B");
});
```

Run: `npm test --workspace apps/engine -- --testPathPattern dispatch-config`  
Expected: FAIL — dli.yaml and markdown.yaml not found.

- [ ] **Step 2: Create dli.yaml**

```yaml
brand:
  name: "DLI, by Studio B"

  voice: |
    Kevin's voice. Analyst/builder register — but Kevin, not a quant terminal.
    He happens to be building a lending index. He's telling you what he found.
    
    Register: Benchmarks / build-in-public
    Tone: Curious, direct, practitioner-honest. Data is evidence, not the whole story.
      Kevin has opinions about what the numbers mean. Not a white paper.
      Not a newsletter robot reciting spread compression. A person with receipts.
    Sentence style: Same short declarative sentences as Dispatch. First person.
      "We found that..." "Here's what the data is telling us." "This surprised me."
    What to include: The number or finding that matters, Kevin's read on what it means,
      one concrete implication for operators or LPs. Build-in-public: what was built,
      what broke, what was learned — told like a war story, not a commit log.
    What to avoid: Academic register ("the data suggests a statistically significant...").
      Pure data dumps with no interpretation. Sentences that sound like a Cliffwater report.
      Quant jargon without a plain-English translation.
    Sample opener: |
      The Q1 SBDLI spread compressed 85bps. That's not a data point — that's a signal
      that something changed in how LMM lenders are pricing risk. Here's what I think
      is happening.
    Length target: 300-600 words for ad hoc issues; 600-900 words for quarterly index issues.
    Output format: Full newsletter issue as prose. For quarterly index issues, one-paragraph
      intro, then 3-4 observations, then one implication paragraph.

  voice_reference: "memory/feedback_studio-b-voice.md"

  audience: |
    LPs considering private credit allocations, credit professionals in LMM lending,
    operators in LMM companies who want to understand the lending environment they're
    borrowing in. People who read the data but want someone's interpretation, not
    just the spreadsheet.

  forbidden:
    - "seamless"
    - "intuitive"
    - "best-in-class"
    - "enterprise-grade"
    - "comprehensive"
    - "statistically significant"
    - "the data suggests"
    - "leverage"
    - "synergy"
    - "AI-powered"
    - "next-gen"
    - "world-class"
    - "we believe"
    - "may help you"

content:
  library_source: ""
  types:
    - newsletter_issue
    - social_thread_x
    - social_post_linkedin

channels:
  beehiiv:
    publication_id: "DLI_PUB_ID"  # replace with pub_e647b558-3a6e-4a29-87ff-c5fd2b566c35
    adapter: beehiiv_draft
    output_type: newsletter_issue
  x_thread:
    adapter: typefully
    output_type: social_thread_x
    posts_in_thread: 4
  linkedin:
    adapter: hubspot
    output_type: social_post_linkedin
    format: text_post

signal_sources:
  quarterly_sbdli:
    description: "Quarterly SBDLI index publish from kbibelhausen/wasala repo — primary DLI trigger"
    adapter: github_release
    repo: "kbibelhausen/wasala"
    event: "release.published"
    trigger_type: full_issue
  wasala_pr_merge:
    description: "Merged PRs in kbibelhausen/wasala — build-in-public DLI content"
    adapter: github_pr
    repo: "kbibelhausen/wasala"
    event: "pull_request.merged"
    min_diff_lines: 50  # skip small fixes
    trigger_type: build_note
  lmm_market_event:
    description: "Credit/LMM discussion on Kevin's LinkedIn — market commentary trigger"
    adapter: linkedin_signal
    brand_filter: "studio-b"
    keyword_filter: ["direct lending", "LMM", "SBDLI", "private credit", "spread"]
    min_signal_score: 60
```

- [ ] **Step 3: Create markdown.yaml**

```yaml
brand:
  name: "Markdown, by Studio B"

  voice: |
    Kevin's voice. Product builder register — direct, specific, shows the work.
    
    Register: Build-in-public / technical operator
    Tone: Kevin built this and he's telling you exactly what happened.
      Not a product announcement. Not a dev blog. A war story from the build.
      Technical details are welcome — but always in service of the story.
    Sentence style: Same short declarative sentences. First person.
      "We shipped X. Here's what broke." "The thing nobody tells you about
      building on Acumatica is..."
    What to include: What was built, why, what broke, what was learned.
      Specific version numbers, architectural decisions, the mistake that cost
      two days. Concrete over abstract.
    What to avoid: Product announcement register ("We're excited to share...").
      Changelog format. Marketing language. Press release or SaaS blog voice.
    Sample opener: |
      We've been running Bolt on Heritage Fabrics' warehouse for six months.
      Here's the part of the stack that keeps breaking — and why we haven't
      fixed it yet.
    Length target: 400-700 words per issue.
    Output format: Full newsletter issue as prose. Start with the incident or
      decision. Show the reasoning. End with the lesson or the open question.

  voice_reference: "memory/feedback_studio-b-voice.md"

  audience: |
    Technical founders, Acumatica ecosystem developers, operators who build
    software on top of their operations. People who can follow the specifics —
    what the error message said, why the migration ran in the wrong schema,
    what the Playwright test was checking.

  forbidden:
    - "seamless"
    - "intuitive"
    - "best-in-class"
    - "enterprise-grade"
    - "comprehensive"
    - "AI-powered"
    - "next-gen"
    - "world-class"
    - "excited to share"
    - "we believe"
    - "game-changing"
    - "revolutionary"
    - "synergy"
    - "leverage"

content:
  library_source: ""
  types:
    - newsletter_issue
    - social_thread_x
    - social_post_linkedin

channels:
  beehiiv:
    publication_id: "MARKDOWN_PUB_ID"  # replace with actual pub ID from docs/pub-ids.md
    adapter: beehiiv_draft
    output_type: newsletter_issue
  x_thread:
    adapter: typefully
    output_type: social_thread_x
    posts_in_thread: 4
  linkedin:
    adapter: hubspot
    output_type: social_post_linkedin
    format: text_post

signal_sources:
  bolt_wms_pr:
    description: "Merged PRs in studio-b-ai/bolt-wms — build-in-public for Bolt"
    adapter: github_pr
    repo: "studio-b-ai/bolt-wms"
    event: "pull_request.merged"
    min_diff_lines: 80
    label_filter: ["feature", "fix"]  # skip chore/deps PRs
    trigger_type: build_note
  acuops_pipeline_pr:
    description: "Merged PRs in studio-b-ai/acuops-pipeline — build-in-public for AcuOps"
    adapter: github_pr
    repo: "studio-b-ai/acuops-pipeline"
    event: "pull_request.merged"
    min_diff_lines: 80
    trigger_type: build_note
  amplify_pr:
    description: "Merged PRs in studio-b-ai/amplify — build-in-public for Amplify"
    adapter: github_pr
    repo: "studio-b-ai/amplify"
    event: "pull_request.merged"
    min_diff_lines: 80
    trigger_type: build_note
  manual_field_note:
    description: "Kevin submits a build note directly via Slack command"
    adapter: slack_intake
    channel: "#raw-content-studiob"
```

- [ ] **Step 4: Replace pub ID placeholders**

Read `docs/pub-ids.md` and replace:
- `DLI_PUB_ID` → `pub_e647b558-3a6e-4a29-87ff-c5fd2b566c35`
- `MARKDOWN_PUB_ID` → actual Markdown pub ID
- `DISPATCH_PUB_ID` in dispatch.yaml (from Task 1) → actual Dispatch pub ID

- [ ] **Step 5: Run tests to verify both configs load**

```bash
cd /Users/kevin/dev/amplify
npm test --workspace apps/engine -- --testPathPattern dispatch-config
```

Expected: All 4 tests PASS (dispatch + dispatch-fields + dli + markdown).

- [ ] **Step 6: Commit**

Invoke `/ship` with message `"feat(brands): add DLI and Markdown brand voice configs"`.

---

## Task 3: beehiiv draft adapter

**Files:**
- Create: `studio-b-ai/amplify/apps/workers/src/adapters/beehiiv-draft.ts`
- Create: `studio-b-ai/amplify/apps/workers/src/adapters/beehiiv-draft.test.ts`

When Amplify generates a newsletter issue, this adapter POSTs it to beehiiv as a draft so Nael can review it in the beehiiv UI before publishing. Uses the beehiiv REST API.

**beehiiv draft creation API:**
```
POST https://api.beehiiv.com/v2/publications/{publicationId}/posts
Authorization: Bearer {BEEHIIV_API_KEY}
Body: {
  "web_title": "string",
  "subtitle": "string",
  "status": "draft",
  "content_tags": ["string"],
  "authors": [{ "id": "string" }],
  "email_body": "<html body>",  // or "free_preview_html"
  "audience": "free"
}
```

The BEEHIIV_API_KEY is already in `consulting-b-studio/.mcp.json`. Add it as an env var on the amplify-workers Railway service.

- [ ] **Step 1: Write failing test**

Create `apps/workers/src/adapters/beehiiv-draft.test.ts`:

```typescript
import { vi, describe, test, expect } from "vitest";
import { BeehiivDraftAdapter } from "./beehiiv-draft.js";

const MOCK_PUB_ID = "pub_test_123";
const MOCK_API_KEY = "test_api_key";

describe("BeehiivDraftAdapter", () => {
  test("creates a draft post via beehiiv API", async () => {
    const fetchSpy = vi.spyOn(globalThis, "fetch").mockResolvedValueOnce({
      ok: true,
      json: async () => ({
        data: { id: "post_abc123", web_title: "Test Issue", status: "draft" }
      }),
    } as Response);

    const adapter = new BeehiivDraftAdapter({
      apiKey: MOCK_API_KEY,
      publicationId: MOCK_PUB_ID,
    });

    const result = await adapter.createDraft({
      title: "Test Issue",
      subtitle: "A test subtitle",
      body: "<p>Test content</p>",
    });

    expect(result.id).toBe("post_abc123");
    expect(result.status).toBe("draft");

    // Verify wire format
    const call = fetchSpy.mock.calls[0];
    const [url, options] = call as [string, RequestInit];
    expect(url).toBe(`https://api.beehiiv.com/v2/publications/${MOCK_PUB_ID}/posts`);
    expect(options.method).toBe("POST");
    expect(JSON.parse(options.body as string)).toMatchObject({
      web_title: "Test Issue",
      subtitle: "A test subtitle",
      status: "draft",
    });
  });

  test("throws on API error response", async () => {
    vi.spyOn(globalThis, "fetch").mockResolvedValueOnce({
      ok: false,
      status: 401,
      text: async () => "Unauthorized",
    } as Response);

    const adapter = new BeehiivDraftAdapter({
      apiKey: "bad_key",
      publicationId: MOCK_PUB_ID,
    });

    await expect(
      adapter.createDraft({ title: "Test", subtitle: "", body: "<p>x</p>" })
    ).rejects.toThrow("beehiiv API error 401");
  });
});
```

Run: `npm test --workspace apps/workers -- --testPathPattern beehiiv-draft`  
Expected: FAIL — beehiiv-draft.ts not found.

- [ ] **Step 2: Implement BeehiivDraftAdapter**

Create `apps/workers/src/adapters/beehiiv-draft.ts`:

```typescript
export interface BeehiivDraftConfig {
  apiKey: string;
  publicationId: string;
}

export interface DraftInput {
  title: string;
  subtitle: string;
  body: string;
  tags?: string[];
}

export interface DraftResult {
  id: string;
  status: string;
  webTitle: string;
  webUrl?: string;
}

export class BeehiivDraftAdapter {
  private readonly apiKey: string;
  private readonly publicationId: string;
  private readonly baseUrl = "https://api.beehiiv.com/v2";

  constructor(config: BeehiivDraftConfig) {
    this.apiKey = config.apiKey;
    this.publicationId = config.publicationId;
  }

  async createDraft(input: DraftInput): Promise<DraftResult> {
    const url = `${this.baseUrl}/publications/${this.publicationId}/posts`;
    const response = await fetch(url, {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${this.apiKey}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        web_title: input.title,
        subtitle: input.subtitle,
        status: "draft",
        content_tags: input.tags ?? [],
        free_preview_html: input.body,
        audience: "free",
      }),
    });

    if (!response.ok) {
      const text = await response.text();
      throw new Error(`beehiiv API error ${response.status}: ${text}`);
    }

    const json = await response.json() as { data: { id: string; status: string; web_title: string; web_url?: string } };
    return {
      id: json.data.id,
      status: json.data.status,
      webTitle: json.data.web_title,
      webUrl: json.data.web_url,
    };
  }
}

export function createBeehiivAdapter(publicationId: string): BeehiivDraftAdapter {
  const apiKey = process.env.BEEHIIV_API_KEY;
  if (!apiKey) throw new Error("BEEHIIV_API_KEY env var required");
  return new BeehiivDraftAdapter({ apiKey, publicationId });
}
```

- [ ] **Step 3: Run tests**

```bash
cd /Users/kevin/dev/amplify
npm test --workspace apps/workers -- --testPathPattern beehiiv-draft
```

Expected: PASS (both tests).

- [ ] **Step 4: Add BEEHIIV_API_KEY to Railway**

The key is `qFOrUw6ZDmBCmnZ5Hka7xNpaYQNCZma1PVajfDNWrp8Z1jMsjxPEzpKh6MRokSrK` (in `.mcp.json`). Add it to the `amplify-workers` Railway service:

```bash
cd /Users/kevin/dev/amplify
railway variables set BEEHIIV_API_KEY=qFOrUw6ZDmBCmnZ5Hka7xNpaYQNCZma1PVajfDNWrp8Z1jMsjxPEzpKh6MRokSrK --service amplify-workers --skip-deploys
```

- [ ] **Step 5: Ship with /ship**

Invoke `/ship` with message `"feat(workers): BeehiivDraftAdapter — POST newsletter drafts to beehiiv"`. The skill runs tests, shows the diff, and creates the PR.

---

## Task 4: Newsletter generation mode in content-worker

**Files:**
- Modify: `studio-b-ai/amplify/apps/workers/src/content-worker.ts`

Extend the content worker to handle `newsletter_issue` content type: when a brand config specifies `channels.beehiiv`, generate a full newsletter issue via Claude and POST it to beehiiv as a draft. Also generate social outputs (X.com thread via Typefully, LinkedIn post) in the same run.

The Slack notification to Kevin (newsletter + social in one message) is the approval gate.

- [ ] **Step 1: Write failing test**

Add to `apps/workers/src/__tests__/content-worker-newsletter.test.ts`:

```typescript
import { vi, describe, test, expect } from "vitest";
import { buildNewsletterJobsForBrand } from "../content-worker.js";

describe("buildNewsletterJobsForBrand", () => {
  test("returns beehiiv job + typefully job + linkedin job for newsletter brand", () => {
    const brand = "dispatch";
    const draft = {
      title: "The audit that found $400K",
      subtitle: "War story from this week",
      body: "<p>We were three days in...</p>",
      xThread: ["We found a $400K discrepancy nobody knew about.", "Here's what it told us."],
      linkedinPost: "Three days into an audit and we found something...",
    };

    const jobs = buildNewsletterJobsForBrand(brand, draft);
    expect(jobs.map(j => j.queue)).toContain(`newsletter-publish-${brand}-beehiiv`);
    expect(jobs.map(j => j.queue)).toContain(`newsletter-publish-${brand}-typefully`);
    expect(jobs.map(j => j.queue)).toContain(`newsletter-publish-${brand}-linkedin`);
  });
});
```

Run: `npm test --workspace apps/workers -- --testPathPattern content-worker-newsletter`  
Expected: FAIL — `buildNewsletterJobsForBrand` not exported.

- [ ] **Step 2: Add `buildNewsletterJobsForBrand` to content-worker.ts**

Add this function to `content-worker.ts`:

```typescript
export interface NewsletterDraft {
  title: string;
  subtitle: string;
  body: string;
  xThread: string[];
  linkedinPost: string;
}

interface NewsletterJobSpec {
  queue: string;
  data: { brand: string; draft: NewsletterDraft };
  delay: number;
}

export function buildNewsletterJobsForBrand(
  brandKey: string,
  draft: NewsletterDraft
): NewsletterJobSpec[] {
  return [
    { queue: `newsletter-publish-${brandKey}-beehiiv`,    data: { brand: brandKey, draft }, delay: 0 },
    { queue: `newsletter-publish-${brandKey}-typefully`,  data: { brand: brandKey, draft }, delay: 0 },
    { queue: `newsletter-publish-${brandKey}-linkedin`,   data: { brand: brandKey, draft }, delay: 0 },
  ];
}
```

- [ ] **Step 3: Run test to verify it passes**

```bash
npm test --workspace apps/workers -- --testPathPattern content-worker-newsletter
```

Expected: PASS.

- [ ] **Step 4: Add newsletter Slack notification function**

The approval notification for newsletter + social is a single Slack message to Kevin. Add to `content-worker.ts`:

```typescript
export function buildNewsletterApprovalMessage(
  brandName: string,
  draft: NewsletterDraft,
  beehiivDraftUrl: string
): string {
  const threadPreview = draft.xThread.slice(0, 2).join("\n\n");
  return (
    `📬 *${brandName}* issue ready — approve to send\n\n` +
    `*"${draft.title}"*\n${draft.subtitle}\n\n` +
    `*beehiiv draft:* ${beehiivDraftUrl}\n\n` +
    `*X thread preview:*\n${threadPreview}...\n\n` +
    `Reply ✅ to approve (sends newsletter + schedules social)\n` +
    `Reply ✏️ to request edits`
  );
}
```

- [ ] **Step 5: Ship with /ship**

Invoke `/ship` with message `"feat(workers): newsletter job builder + Slack approval message for newsletter brands"`.

---

## Task 5: GitHub signal source wiring in webhook-router

**Files:**
- Modify: `studio-b-ai/webhook-router` — add GitHub PR merged → Amplify signal route for the three signal repos

When a PR merges in `studio-b-ai/bolt-wms`, `studio-b-ai/acuops-pipeline`, `studio-b-ai/amplify`, `kbibelhausen/wasala` — emit a signal to the Amplify workers signal queue so the newsletter generation worker picks it up.

- [ ] **Step 1: Check if GitHub webhook handler already exists in webhook-router**

```bash
grep -rn "pull_request\|github.*merge\|pr.*merged" /Users/kevin/dev/webhook-router/src --include="*.ts" | head -20
```

If a handler already exists, read it and extend. If not, proceed with creating a minimal handler.

- [ ] **Step 2: Add GitHub PR merged signal emitter**

Find the GitHub webhook route file in webhook-router. Add or extend the handler to emit Amplify signals on PR merge events:

```typescript
// In existing GitHub webhook handler (or new file):
const NEWSLETTER_SIGNAL_REPOS: Record<string, string> = {
  "studio-b-ai/bolt-wms":      "markdown",
  "studio-b-ai/acuops-pipeline": "markdown",
  "studio-b-ai/amplify":       "markdown",
  "kbibelhausen/wasala":        "dli",
};

export async function handleGithubPrMerged(
  repoFullName: string,
  prNumber: number,
  prTitle: string,
  prBody: string,
  diffLines: number,
  redis: Redis
): Promise<void> {
  const brandKey = NEWSLETTER_SIGNAL_REPOS[repoFullName];
  if (!brandKey) return;  // repo not mapped to a newsletter brand

  const MIN_DIFF_LINES = 50;
  if (diffLines < MIN_DIFF_LINES) return;  // skip small chore PRs

  await redis.lpush("amplify:newsletter-signals", JSON.stringify({
    brand: brandKey,
    source: "github_pr",
    repo: repoFullName,
    pr_number: prNumber,
    pr_title: prTitle,
    pr_body_excerpt: prBody.slice(0, 500),
    diff_lines: diffLines,
    timestamp: new Date().toISOString(),
  }));
}
```

- [ ] **Step 3: Add test for signal emitter**

```typescript
test("handleGithubPrMerged emits to dli queue for wasala repo", async () => {
  const redisMock = { lpush: vi.fn().mockResolvedValue(1) };
  await handleGithubPrMerged(
    "kbibelhausen/wasala", 170, "feat: BDC investment date extraction", "...", 200,
    redisMock as unknown as Redis
  );
  expect(redisMock.lpush).toHaveBeenCalledWith(
    "amplify:newsletter-signals",
    expect.stringContaining('"brand":"dli"')
  );
});

test("handleGithubPrMerged skips small PRs", async () => {
  const redisMock = { lpush: vi.fn() };
  await handleGithubPrMerged(
    "studio-b-ai/bolt-wms", 100, "chore: bump deps", "", 10,
    redisMock as unknown as Redis
  );
  expect(redisMock.lpush).not.toHaveBeenCalled();
});
```

Run tests, verify pass, then commit.

- [ ] **Step 4: Commit**

```bash
Invoke `/ship` in the `webhook-router` repo with message `"feat: GitHub PR merged → Amplify newsletter signal emitter (dli/markdown brands)"`.

---

## Task 6: Amplify-Content Claude Code skill

**Files:**
- Create: `~/.claude/skills/amplify-content/skill.md`

Amplify-Content is a Claude Code skill Kevin invokes in any session (`/amplify-content`) to start a dialogue-driven content creation session. It loads context from Qdrant (recent sessions), prompts Kevin with one probing question at a time, and outputs a full newsletter issue + social thread from the conversation.

- [ ] **Step 1: Create the skill directory and skill.md**

```bash
mkdir -p ~/.claude/skills/amplify-content
```

Create `~/.claude/skills/amplify-content/skill.md`:

```markdown
# Amplify-Content — Dialogue-Driven Newsletter Creation

Use this skill when Kevin invokes `/amplify-content` to create a newsletter issue
through dialogue rather than waiting for a signal to be detected.

## How to start

When invoked, immediately:

1. **Load recent session context from Qdrant:**

```bash
export QDRANT_URL=$(cd ~/dev/acudev && railway variables --service acudev --json 2>/dev/null | python3 -c "import sys,json; print(json.load(sys.stdin)['QDRANT_URL'])")
export VOYAGE_API_KEY=$(cd ~/dev/acudev && railway variables --service acudev --json 2>/dev/null | python3 -c "import sys,json; print(json.load(sys.stdin)['VOYAGE_API_KEY'])")
```

Search Qdrant for "recent work shipped or completed" — last 7 days:
```bash
python3 -c "
import requests, json, os
resp = requests.post('$QDRANT_URL/collections/studiob-sessions/points/search',
    json={'vector': ..., 'limit': 8, 'score_threshold': 0.3,
          'with_payload': True, 'filter': {'must': [{'range': {'key': 'date', 'gte': '7_days_ago'}}]}})
..."
```

2. **Ask Kevin which newsletter this is for:**

   "Dispatch (consulting war story), DLI (index build / market event), or Markdown (product build)?"

   Wait for the answer before asking anything else.

3. **Ask ONE probing question based on recent context + chosen newsletter.**

   For Dispatch: "You worked on [X from recent sessions] this week. What's the moment where you knew it was — or wasn't — going to work?"
   
   For DLI: "The last wasala PR I see is [PR title]. What did you learn from that build that you didn't expect?"
   
   For Markdown: "You shipped [PR title] — what broke first?"

4. **Continue the dialogue, one question at a time.**

   The goal is to get Kevin to tell the story: incident, decision, consequence, lesson. Each question should pull one more specific detail out. Stop when you have a complete story arc.

5. **Draft the newsletter issue.**

   When the dialogue has enough material (typically 5-8 exchanges), say: "I have enough for the issue. Let me draft it."
   
   Draft the full issue in Kevin's voice per the voice config for the chosen newsletter:
   - Dispatch: 400-700 words, operator war story register
   - DLI: 300-600 words, practitioner-honest data + opinion
   - Markdown: 400-700 words, build war story, shows the work
   
   Format: prose, no headers, no bullets, first person throughout.

6. **Draft the social outputs alongside.**

   X.com thread (4 posts):
   - Post 1: opening hook (the sharpest single sentence from the issue)
   - Posts 2-3: key points (two most important details)
   - Post 4: subscribe CTA ("Full dispatch at dispatch.b.studio" or equivalent)
   
   LinkedIn post: slightly longer form, same substance, same voice.

7. **Present all three outputs together.**

   Format:
   ```
   ## Newsletter: [Title]
   [Full issue prose]
   
   ---
   ## X thread
   [Tweet 1]
   [Tweet 2]
   [Tweet 3]
   [Tweet 4]
   
   ---
   ## LinkedIn
   [Post text]
   ```

8. **Ask if Kevin wants revisions or is ready to push to beehiiv draft.**

   If ready: use the beehiiv MCP tool to create a draft post in the appropriate publication.
   If revisions: apply them and re-present the changed section only.

## Voice rules (all three newsletters)

Same voice, different register. All eight rules from `memory/feedback_studio-b-voice.md` apply. Reload it if any doubt arises.

1. State opinions plainly.
2. Specificity beats superlatives.
3. Name the specific system, version, error message.
4. Direct second-person or first-person.
5. Short sentences, short paragraphs.
6. Explain the reasoning, not just the decision.
7. Opinionated — Kevin has a take on what this means.
8. Anti-hype test: would this appear on a generic SaaS landing page? If yes, cut it.

## Signal: when to ask a clarifying question vs. when to draft

Draft when Kevin has described:
- A concrete incident (something happened)
- A decision and the reasoning behind it
- An outcome (what actually resulted)
- One thing that surprised him

If any of these is missing, ask one more focused question before drafting.

## Notes

- This skill runs in whatever session Kevin invokes it from. The output (newsletter draft, social posts) is presented in the session and can be pushed to beehiiv via MCP.
- Phase 2 (Amplify M2 Console): this skill becomes a Console feature with ElevenLabs voice integration for the producer character.
- The podcast format: Kevin can record himself answering these same questions — his real answers from this session become the episode script. No additional prep required.
```

- [ ] **Step 2: Verify skill is discoverable**

```bash
ls ~/.claude/skills/amplify-content/skill.md
```

Expected: File exists.

- [ ] **Step 3: Run one test session**

Invoke `/amplify-content` in a Claude Code session. Confirm:
1. Qdrant context loads (or gracefully falls back if unavailable)
2. Newsletter selector question is asked
3. Probing questions are specific to recent work
4. Draft is produced in correct voice and format
5. Social outputs are generated alongside

Note any issues and update `skill.md` before Task 3.

---

## Task 7: Typefully integration for X.com thread scheduling

**What:** The Typefully MCP is already in `.mcp.json`. When Amplify generates social outputs for a newsletter brand, the X.com thread goes to Typefully as a draft. Nael reviews in Typefully. Kevin approves in the same Slack message as the newsletter.

This task verifies the existing Typefully MCP connection and documents the workflow.

- [ ] **Step 1: Verify Typefully MCP connection**

Test by creating a draft thread via Typefully MCP:

```
Use Typefully MCP: create_draft
content: "Testing Amplify → Typefully connection. Delete this."
threadify: true
```

Expected: Draft appears in Typefully dashboard. Retrieve the draft URL.

- [ ] **Step 2: Delete the test draft**

Via Typefully dashboard or MCP: delete the test draft.

- [ ] **Step 3: Document the Typefully workflow**

Write to `docs/typefully-workflow.md` in consulting-b-studio:

```markdown
# Typefully Workflow — Amplify X.com Thread Scheduling

Amplify generates X.com threads alongside each newsletter issue. The thread
goes to Typefully as a draft. Nael reviews alongside the newsletter. Kevin
approves both in the same Slack message.

**Draft creation:** Via Typefully MCP `create_draft` with `threadify: true`
**Review:** Nael reviews at app.typefully.com
**Publish:** Typefully schedules to Kevin's X.com account at optimal time
**Kevin's approval Slack message includes:** newsletter beehiiv draft link + X thread preview

Typefully API key: stored in consulting-b-studio/.mcp.json
```

Invoke `/ship` in the `consulting-b-studio` repo with message `"docs: Typefully workflow for Amplify social scheduling"`.

---

## Self-Review

**Spec coverage check:**
- ✅ `dispatch` brand voice config with full voice spec (Task 1)
- ✅ `dli` brand voice config with full voice spec (Task 2)
- ✅ `markdown` brand voice config with full voice spec (Task 2)
- ✅ Signal sources: LinkedIn for Dispatch, GitHub PR + quarterly for DLI, GitHub PRs for Markdown (Tasks 1-2 configs + Task 5 signal emitter)
- ✅ beehiiv draft adapter — POSTs generated issues to beehiiv (Task 3)
- ✅ Social content output (X.com thread via Typefully + LinkedIn) alongside each newsletter issue (Task 4)
- ✅ Slack notification to Kevin — newsletter + social in one approval message (Task 4)
- ✅ Amplify-Content dialogue skill (Task 6)
- ✅ Typefully integration verified (Task 7)

**Phase 2 (NOT in this plan — deferred to Amplify M2):**
- ElevenLabs producer voice integration for podcast episodes
- Full recording workflow (producer audio + Kevin audio)
- YouTube channel + episode publishing
- AI video avatar for producer persona

**Voice configs are additive only:** Adding `dispatch.yaml`, `dli.yaml`, `markdown.yaml` does not affect existing `studio-b.yaml` or `aesthetik.yaml` configs. Existing LinkedIn content generation for Studio B / Aesthetik is unaffected.
