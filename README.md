# Lanbow Claw Skill

AI-native Meta Ads workflow skill for Openclaw — from market research to campaign optimization, all through natural language.

**[Lanbow](https://lanbow.com/)** | **[Blog: Lanbow Claw Skill](https://lanbow.com/blog/lanbow-claw-skill)**

## What is this?

Lanbow Claw Skill is an Openclaw skill that turns your AI assistant into a full-stack Meta advertising operator. It covers the complete ad lifecycle in 4 features:

```
Strategy Research → Creative Generation → Ad Delivery → Post-Campaign Review
       ↑                                                        │
       └────────────────── optimization loop ───────────────────┘
```

| Feature                  | What it does                                                  | Powered by                      |
| ------------------------ | ------------------------------------------------------------- | ------------------------------- |
| **Strategy Research**    | Competitive analysis, audience insights, messaging framework  | Web Search + Web Fetch          |
| **Creative Generation**  | AI-generated ad images with visual layouts and copy           | Google Gemini API               |
| **Ad Delivery**          | Campaign creation, targeting, budgets, media upload           | lanbow-ads CLI + Meta Graph API |
| **Post-Campaign Review** | Performance diagnosis, root cause analysis, optimization plan | lanbow-ads CLI insights data    |

## Quickstart

### 1. Install the skill

Tell your AI coding agent to read the installation guide:

> "Read https://raw.githubusercontent.com/sandwichlab-ai/lanbow-claw-skill/main/INSTALL.md and follow the instructions to install this skill."

**Important:** The skill requires the `lanbow-ads` CLI to work. Make sure to install it alongside the skill files:

```bash
npm install -g lanbow-ads
```

### 2. Configure dependencies

Each feature has its own API dependency — you only need the ones you use:

```bash
# Feature 1 (Strategy): No extra config — uses built-in WebSearch/WebFetch

# Feature 2 (Creative): Set your Gemini API key
export GEMINI_API_KEY="your-google-gemini-api-key"

# Feature 3 (Ad Delivery): Install and auth the CLI
npm install -g lanbow-ads                # Install CLI globally
lanbow-ads auth login                    # Opens browser for OAuth
lanbow-ads config set --account act_XXX  # Set default ad account

# Feature 4 (Post-Campaign Review): Uses the same CLI as Feature 3
```

### 3. Start using it

Talk to Openclaw naturally — the skill triggers automatically:

```
"Help me plan a Meta ads strategy for my e-commerce store example.com"
→ Triggers Feature 1: Strategy Research

"Generate 3 ad creatives for this product, targeting young professionals"
→ Triggers Feature 2: Creative Generation

"Create a campaign with $50/day budget targeting US 25-45 year olds"
→ Triggers Feature 3: Ad Delivery

"Review the performance of campaign 123456 and tell me what to optimize"
→ Triggers Feature 4: Post-Campaign Review
```

## Use Cases

### Run a full campaign from zero

```
You: "I want to run Meta ads for my skincare brand skinlab.com.
      Target: US women 25-40. Budget: $100/day. Goal: website purchases."
```

Claude will:
1. Research your market, competitors, and audience via web search
2. Build a messaging strategy (key message, key look, ad angles)
3. Generate ad creative images using Gemini
4. Create the full campaign structure (campaign → ad set → creative → ad) via CLI
5. Set everything to PAUSED for your review before going live

### Optimize a running campaign

```
You: "My campaign 120330123456 has been running for a week. Review it and suggest next steps."
```

Claude will:
1. Pull insights data across multiple dimensions (daily trend, ad set, audience breakdowns)
2. Run a metric health check (CTR, CPC, CPM, frequency, ROAS)
3. Diagnose root causes (creative fatigue? audience saturation? targeting mismatch?)
4. Deliver a prioritized optimization plan (P0 immediate → P1 creative → P2 targeting → P3 strategic)

### Refresh creatives based on performance

```
You: "The review says creative fatigue. Generate 3 new ad images with a fresh angle."
```

Claude will pick up context from the previous review, generate new proposals and images, and help you swap them into the running campaign.

### Strategy-only or delivery-only

Each feature works independently:

```
"Research competitor ad strategies for protein powder brands on Meta"
→ Strategy research only — no campaign created

"Upload this image and create a creative with headline 'Shop Now'"
→ Ad delivery only — no strategy needed
```

## Skill Structure

```
lanbow-claw-skill/
├── SKILL.md                              # Main orchestrator (workflow + feature overview)
└── references/
    ├── strategy-research.md              # Feature 1: full research workflow
    ├── strategy-template.md              # Report section structure
    ├── strategy-meta-only-template.md    # Meta-only strategy template
    ├── strategy-document-standards.md    # Report formatting standards
    ├── creative-generation.md            # Feature 2: Gemini pipeline + prompts
    ├── ad-delivery.md                    # Feature 3: lanbow-ads CLI guide
    ├── ad-delivery-commands.md           # Complete CLI command reference
    ├── meta-account-setup.md            # Meta developer & ad account setup guide
    └── post-campaign-review.md           # Feature 4: diagnostic framework
```

## Roadmap

### Now (v0.1)
- [x] Strategy research via web search (Meta-only template)
- [x] Creative generation via Gemini 2-step pipeline
- [x] Full campaign CRUD via lanbow-ads CLI (campaigns, ad sets, ads, creatives, images, videos)
- [x] Post-campaign review with metric benchmarks and diagnostic framework
- [x] Self-contained skill packaging (all references bundled)

### Next
- [ ] A/B test design — auto-generate test structures (creative variants, audience splits) from review insights
- [ ] Budget optimizer — suggest daily budget adjustments based on pacing and ROAS trends
- [ ] Audience expansion — auto-discover new interest/lookalike targets from top-performing segments
- [ ] Multi-campaign coordination — manage multiple campaigns as a portfolio with cross-campaign budget allocation
- [ ] Automated reporting — scheduled performance summaries with trend charts

### Future
- [ ] Omni-channel strategy support (Amazon + TikTok + Meta)
- [ ] Video creative generation pipeline
- [ ] Landing page analysis integration (post-click optimization)
- [ ] Custom benchmark library — learn from your own historical data instead of industry averages
- [ ] Webhook-triggered auto-review — kick off post-campaign review when performance anomalies are detected
