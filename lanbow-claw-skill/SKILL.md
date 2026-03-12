---
name: lanbow-ads-skills
description: |
  End-to-end Meta (Facebook/Instagram) advertising system orchestrator covering the full ad lifecycle:
  strategy research, creative generation, campaign delivery, and post-campaign optimization.
  Use when:
  (1) Planning and executing a complete Meta Ads campaign from scratch
  (2) Running the full ads lifecycle: research → create → launch → review → optimize
  (3) User says "run ads", "launch campaign", "full ads system", "ads pipeline", "广告投放全流程"
  (4) User needs guidance on which ads skill to use for their current task
  (5) Coordinating between strategy, creative, delivery, and review phases
  This skill orchestrates 4 sub-skills: ads-strategy-researcher (insights), creative_gen (creatives),
  lanbow-ads (delivery), and post-campaign-review (optimization). Each can also be used independently.
homepage: https://github.com/sandwichlab-ai/lanbow-claw-skill
metadata:
  {
    "openclaw":
      {
        "emoji": "📢",
        "requires": {
          "bins": ["lanbow-ads"],
          "env": ["META_ACCESS_TOKEN", "META_APP_SECRET", "META_APP_ID", "META_AD_ACCOUNT_ID", "GEMINI_API_KEY"]
        },
        "primaryEnv": "META_ACCESS_TOKEN"
      }
  }
---

# Lanbow Ads Skills

End-to-end Meta Ads lifecycle management across 4 features: **Strategy → Creative → Delivery → Review**, forming a continuous optimization loop.

## Feature Map

| # | Feature | Skill | What It Does | Key Dependency |
|---|---------|-------|------------|----------------|
| 1 | Strategy Research | `ads-strategy-researcher` | Market analysis, competitive intelligence, messaging strategy | WebSearch / WebFetch |
| 2 | Creative Generation | `creative_gen` | AI-generated ad images from strategy inputs | User's Gemini API Key |
| 3 | Ad Delivery | `lanbow-ads` | Campaign creation and management via Meta Ads CLI | User's Meta Access Token |
| 4 | Post-Campaign Review | `post-campaign-review` | Performance diagnosis and optimization plan | Campaign delivery data |

## System Decision Tree

**Starting a new campaign from scratch?**
→ Start at Feature 1 (Strategy) → Feature 2 (Creative) → Feature 3 (Delivery)

**Have strategy, need creatives?**
→ Start at Feature 2 (Creative) → Feature 3 (Delivery)

**Have creatives, need to launch?**
→ Start at Feature 3 (Delivery)

**Campaign already running, need to optimize?**
→ Start at Feature 4 (Review) → loop back to Features 1-3 as needed

**Quick ad hoc task?** (list campaigns, check performance, upload image, etc.)
→ Use Feature 3 (Delivery / lanbow-ads CLI) directly

## Full Lifecycle Loop

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Feature 1: Strategy Research                           │
│  ┌──────────────────────────────────┐                   │
│  │ Input: URL, objectives, audience │                   │
│  │ Output: insights report          │                   │
│  │   • key message                  │                   │
│  │   • key look                     │                   │
│  │   • ad angles + CTAs             │                   │
│  │   • audience segments            │                   │
│  └──────────┬───────────────────────┘                   │
│             │                                           │
│             ▼                                           │
│  Feature 2: Creative Generation                         │
│  ┌──────────────────────────────────┐                   │
│  │ Input: key message, key look,    │                   │
│  │        product info, audiences   │                   │
│  │ Output: ad images + prompts      │                   │
│  └──────────┬───────────────────────┘                   │
│             │                                           │
│             ▼                                           │
│  Feature 3: Ad Delivery                                 │
│  ┌──────────────────────────────────┐                   │
│  │ Input: creatives, targeting,     │                   │
│  │        budget, account           │                   │
│  │ Output: live campaign            │                   │
│  └──────────┬───────────────────────┘                   │
│             │                                           │
│             ▼                                           │
│  Feature 4: Post-Campaign Review                        │
│  ┌──────────────────────────────────┐                   │
│  │ Input: performance data          │                   │
│  │ Output: diagnosis + next actions │──── loop back ────┘
│  └──────────────────────────────────┘
```

## Feature 1: Strategy Research

**Skill:** `ads-strategy-researcher` | **Details:** [strategy-research.md](references/strategy-research.md)

Perform pre-campaign market analysis using WebSearch + WebFetch. Produces a structured strategy report.

**Trigger phrases:** "ads strategy", "marketing plan", "competitor research", "market research"

**Inputs:** product URL, business objectives, campaign context, audience info, competitors

**Key outputs that feed Feature 2:**

| Output | Used By |
|--------|---------|
| Key Message (primary promise + reasons) | `product_info` / `requirements` in creative gen |
| Key Look (visual cues, composition) | `requirements` in creative gen |
| Ad Angles (hooks, CTAs) | `input_cta` in creative gen |
| Audience Segments | `audience_descriptions` in creative gen |

**Strategy types:** Omni-Channel (Amazon + TikTok + Meta) or Meta-Only

## Feature 2: Creative Generation

**Skill:** `creative_gen` | **Details:** [creative-generation.md](references/creative-generation.md)

Generate ad creative images via 2-step Gemini pipeline: proposals (gemini-2.5-flash) → images (gemini-3.1-flash-image-preview).

**Trigger phrases:** "generate ad", "create ad image", "ad creative", "generate creatives"

**Requires:** `GEMINI_API_KEY` from user

**Inputs:** product info, audience descriptions, requirements, CTA, aspect ratio, optional product image

**Outputs:** `creative-proposals.json` + `ad-*.png` images + reusable prompt templates

**Connection to Feature 3:** Upload generated images via `lanbow-ads images upload`, then reference image_hash in `lanbow-ads creatives create`.

## Feature 3: Ad Delivery

**Skill:** `lanbow-ads` | **Details:** [ad-delivery.md](references/ad-delivery.md)

Execute campaign setup and management via the `lanbow-ads` CLI.

**Trigger phrases:** "create campaign", "launch ads", "upload image", "show campaigns", "投放广告"

**Requires:** Meta Access Token, App ID, App Secret, Ad Account ID (declared in `requires.env` as `META_ACCESS_TOKEN`, `META_APP_SECRET`, `META_APP_ID`, `META_AD_ACCOUNT_ID`)

**Credential setup (two methods):**

1. **Environment variables (preferred):** If `META_ACCESS_TOKEN`, `META_APP_ID`, `META_APP_SECRET`, and `META_AD_ACCOUNT_ID` are already set as environment variables, configure the CLI automatically:
   ```bash
   lanbow-ads config set --app-id "$META_APP_ID" --app-secret "$META_APP_SECRET"
   lanbow-ads auth set-token "$META_ACCESS_TOKEN"
   lanbow-ads config set --account "$META_AD_ACCOUNT_ID"
   ```

2. **Interactive setup (fallback):** If env vars are not set, guide the user through the setup. Ask for their App ID, App Secret, Access Token, and Ad Account ID, then run the CLI commands to configure. For a full walkthrough, see [meta-account-setup.md](references/meta-account-setup.md).

**Credential scope limits:** The agent must ONLY use provided credentials for `lanbow-ads` CLI commands. The agent must NOT log, echo, or store credentials in any file other than via the `lanbow-ads config set` and `lanbow-ads auth set-token` commands. The agent must NOT transmit credentials to any endpoint other than the Meta Marketing API (via the CLI).

**Campaign creation sequence:**
1. `lanbow-ads campaigns create` — create campaign (PAUSED)
2. `lanbow-ads images upload` / `lanbow-ads videos upload` — upload media
3. `lanbow-ads creatives create` — create creative with media
4. `lanbow-ads adsets create` — create ad set with targeting + budget
5. `lanbow-ads ads create` — create ad linking ad set + creative
6. Verify with `lanbow-ads campaigns get` / `lanbow-ads ads get`

**Key rules:**
- Budgets in **cents** (5000 = $50.00)
- Always create in **PAUSED** status, activate after review
- Use `--json` flag when output feeds into another step
- For OUTCOME_SALES: must include `--promoted-object` with pixel_id

## Feature 4: Post-Campaign Review

**Skill:** `post-campaign-review` | **Details:** [post-campaign-review.md](references/post-campaign-review.md)

Analyze delivery data, diagnose performance issues, generate optimization recommendations.

**Trigger phrases:** "review campaign", "campaign diagnosis", "optimize ads", "复盘", "投后分析"

**Data collection:** Fetch via `lanbow-ads insights get --json` at multiple dimensions (campaign, ad set, ad, audience breakdowns, daily trends).

**Diagnostic framework:**
- Metric health check against benchmarks (CTR, CPC, CPM, frequency, ROAS)
- Root cause patterns: creative fatigue, audience saturation, targeting mismatch, budget issues, landing page problems

**Output:** Prioritized optimization plan (P0 immediate → P1 creative → P2 targeting → P3 strategic)

**Feedback loop:** Review findings feed back into:
- Feature 1 (new strategy if targeting mismatch or new opportunity found)
- Feature 2 (creative refresh if fatigue diagnosed)
- Feature 3 (budget/targeting adjustments, pause/activate ads)

## Cross-Feature Data Flow

```
Strategy Report ──→ key message, key look, audiences ──→ Creative Gen
Creative Gen   ──→ ad images, proposals             ──→ Ad Delivery (upload + create)
Ad Delivery    ──→ live campaign + performance data  ──→ Post-Campaign Review
Review         ──→ optimization actions              ──→ Ad Delivery (adjust)
Review         ──→ creative refresh needs            ──→ Creative Gen (new round)
Review         ──→ strategy refinement needs         ──→ Strategy Research (re-analyze)
```

## Security & Privacy

### Required Credentials (all declared in `metadata.openclaw.requires.env`)

| Env Var | Sensitivity | Used By | Storage Location |
|---------|------------|---------|-----------------|
| `META_ACCESS_TOKEN` | **High** — grants full ad account access | Feature 3 (Ad Delivery) | `lanbow-ads` CLI config dir (`~/.config/lanbow-ads/`) |
| `META_APP_SECRET` | **High** — can generate long-lived tokens | Feature 3 (Ad Delivery) | `lanbow-ads` CLI config dir |
| `META_APP_ID` | Medium — app identifier | Feature 3 (Ad Delivery) | `lanbow-ads` CLI config dir |
| `META_AD_ACCOUNT_ID` | Low — account identifier | Feature 3 (Ad Delivery) | `lanbow-ads` CLI config dir |
| `GEMINI_API_KEY` | **High** — API access | Feature 2 (Creative Gen) | User's environment only; never persisted by this skill |

### Required Runtime Dependencies

| Dependency | Source | Purpose |
|-----------|--------|---------|
| `lanbow-ads` CLI | Install via `npm i -g lanbow-ads` ([npm registry](https://www.npmjs.com/package/lanbow-ads)) | Campaign management and performance data retrieval via Meta Marketing API |
| WebSearch / WebFetch | Built-in agent tools | Market research and competitive intelligence gathering |

### Credential Scope Restrictions

The agent is restricted to the following credential operations **only**:
- `lanbow-ads config set --app-id` / `--app-secret` / `--account` — configure CLI
- `lanbow-ads auth set-token` — set access token
- `lanbow-ads auth exchange` — exchange short-lived token for long-lived token
- `lanbow-ads auth status` — verify token validity

The agent must **NOT**:
- Log, echo, or print credentials in plain text
- Store credentials in any file other than via `lanbow-ads` CLI commands
- Transmit credentials to any endpoint other than the Meta Marketing API (via CLI)
- Use credentials for purposes outside ad campaign management

### Data Flow & Privacy

- **What leaves the machine:** API calls to Meta Marketing API (via `lanbow-ads` CLI), API calls to Google Gemini (for image generation), web searches (for strategy research)
- **What stays local:** Strategy reports, generated creative images, campaign review documents, CLI configuration files
- **Credential handling:** Credentials are configured via the `lanbow-ads` CLI and stored in its config directory (`~/.config/lanbow-ads/`). This skill itself does not persist or transmit credentials beyond passing them to the CLI.

### Token Lifetime & Cleanup

| Token Type | Lifetime | When to Use |
|-----------|----------|-------------|
| User Access Token (Graph API Explorer) | ~1-2 hours | Quick tests, one-off campaigns |
| Long-lived User Token (via `auth exchange`) | ~60 days | Short-term automation |
| System User Token | Never expires | Production/agency use only |

**After use, always clean up stored credentials:**
```bash
lanbow-ads auth logout          # Remove stored access token
lanbow-ads config unset --app-secret  # Remove stored app secret
lanbow-ads config list          # Verify no secrets remain
```

### Security Recommendations

1. **Prefer short-lived tokens** — start with a User Access Token from Graph API Explorer (~1-2 hours). Only use System User Tokens for production automation where you control the runtime
2. **Verify your runtime environment** — only provide credentials if the agent runs on a machine you trust. Do NOT paste secrets into hosted/shared agent environments
3. **Use a dedicated test Ad Account** — create a separate Ad Account with minimal budget for agent-managed campaigns; do not use your primary production account
4. **Least privilege scopes** — when generating tokens, grant only `ads_management` and `ads_read` unless additional permissions are explicitly needed
5. **Rotate after use** — revoke tokens in [Meta Business Settings](https://business.facebook.com/settings/) after your session ends, especially if you used long-lived tokens or App Secret
6. **Verify `lanbow-ads` provenance** — install from the [official npm registry](https://www.npmjs.com/package/lanbow-ads) and verify the package before use

## Resources

All sub-skill instructions and reference materials are self-contained in `references/`.

### Feature 1: Strategy Research
- **[strategy-research.md](references/strategy-research.md)** — Full strategy research system, research protocol, chapter-based output rules
- **[strategy-template.md](references/strategy-template.md)** — Report section structure (Executive Summary → Controls)
- **[strategy-meta-only-template.md](references/strategy-meta-only-template.md)** — Meta-only campaign strategy template
- **[strategy-document-standards.md](references/strategy-document-standards.md)** — Report formatting standards, table numbering, data source annotations

### Feature 2: Creative Generation
- **[creative-generation.md](references/creative-generation.md)** — Full Gemini 2-step pipeline: proposals → image generation, API calls, prompt templates

### Feature 3: Ad Delivery
- **[ad-delivery.md](references/ad-delivery.md)** — Full lanbow-ads CLI guide: campaign creation, targeting, insights, configuration
- **[ad-delivery-commands.md](references/ad-delivery-commands.md)** — Complete command reference with all flags, types, and descriptions
- **[meta-account-setup.md](references/meta-account-setup.md)** — Step-by-step Meta developer registration, app creation, OAuth setup, and ad account configuration

### Feature 4: Post-Campaign Review
- **[post-campaign-review.md](references/post-campaign-review.md)** — Full diagnostic framework, metric benchmarks, optimization plan template, feedback loop
