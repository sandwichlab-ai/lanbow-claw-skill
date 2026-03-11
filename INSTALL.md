# Installation Guide

Lanbow Claw Skill works with the following AI coding agents:

| Platform | Skill Mechanism | Installation Method |
|----------|----------------|---------------------|
| **OpenClaw** | Multi-level skill directories + ClawHub registry | ClawHub / manual copy |
| **Claude Code** | `~/.claude/skills/` directory | Manual copy |

---

## Platform 1: OpenClaw

OpenClaw loads skills from multiple locations. When skill names conflict, higher-priority locations win:

```
workspace skills       <workspace>/skills/                     (highest priority)
project agent skills   <workspace>/.agents/skills/
personal agent skills  ~/.agents/skills/
managed skills         ~/.openclaw/skills/
bundled skills         (shipped with OpenClaw, read-only)
extra dirs             (configured in openclaw.json)            (lowest priority)
```

> The base config directory (`~/.openclaw`) can be overridden via the `OPENCLAW_STATE_DIR` environment variable.

Choose an install location based on your needs:

| Scope | Path (macOS / Linux) | Path (Windows) | When to use |
|-------|---------------------|----------------|-------------|
| **This project only** | `<workspace>/skills/` | `<workspace>\skills\` | You only need the skill in one project |
| **All projects for you** | `~/.agents/skills/` | `%USERPROFILE%\.agents\skills\` | Personal skill, shared across all agents |
| **All users on this machine** | `~/.openclaw/skills/` | `%USERPROFILE%\.openclaw\skills\` | Default `clawhub install` location |

### Option A: Install via ClawHub

If the skill is published to [ClawHub](https://github.com/openclaw/clawhub):

```bash
# Installs to ~/.openclaw/skills/ (managed, visible to all agents)
clawhub install lanbow-ads-skills
```

### Option B: Manual Install

```bash
# 1. Clone the repo
git clone https://github.com/sandwichlab-ai/lanbow-claw-skill.git /tmp/lanbow-claw-skill
```

Then copy skill files to your preferred location:

**Managed install (all agents on this machine):**

```bash
# macOS / Linux
mkdir -p ~/.openclaw/skills/lanbow-ads-skills/references
cp /tmp/lanbow-claw-skill/lanbow-claw-skill/SKILL.md ~/.openclaw/skills/lanbow-ads-skills/
cp /tmp/lanbow-claw-skill/lanbow-claw-skill/references/*.md ~/.openclaw/skills/lanbow-ads-skills/references/
```

```powershell
# Windows PowerShell
$dest = "$env:USERPROFILE\.openclaw\skills\lanbow-ads-skills"
New-Item -ItemType Directory -Path "$dest\references" -Force
Copy-Item /tmp/lanbow-claw-skill/lanbow-claw-skill/SKILL.md -Destination $dest
Copy-Item /tmp/lanbow-claw-skill/lanbow-claw-skill/references/*.md -Destination "$dest\references"
```

**Personal agent install (shared across all agents for current user):**

```bash
# macOS / Linux
mkdir -p ~/.agents/skills/lanbow-ads-skills/references
cp /tmp/lanbow-claw-skill/lanbow-claw-skill/SKILL.md ~/.agents/skills/lanbow-ads-skills/
cp /tmp/lanbow-claw-skill/lanbow-claw-skill/references/*.md ~/.agents/skills/lanbow-ads-skills/references/
```

```powershell
# Windows PowerShell
$dest = "$env:USERPROFILE\.agents\skills\lanbow-ads-skills"
New-Item -ItemType Directory -Path "$dest\references" -Force
Copy-Item /tmp/lanbow-claw-skill/lanbow-claw-skill/SKILL.md -Destination $dest
Copy-Item /tmp/lanbow-claw-skill/lanbow-claw-skill/references/*.md -Destination "$dest\references"
```

**Project-level install (this workspace only, highest priority):**

```bash
# Run from your project root
mkdir -p skills/lanbow-ads-skills/references
cp /tmp/lanbow-claw-skill/lanbow-claw-skill/SKILL.md skills/lanbow-ads-skills/
cp /tmp/lanbow-claw-skill/lanbow-claw-skill/references/*.md skills/lanbow-ads-skills/references/
```

### Gemini API Key via OpenClaw Config (Optional)

OpenClaw can inject environment variables for a skill via `~/.openclaw/openclaw.json`, so you don't need to manually export them:

```json
{
  "skills": {
    "entries": {
      "lanbow-ads-skills": {
        "enabled": true,
        "env": {
          "GEMINI_API_KEY": "your-gemini-api-key"
        }
      }
    }
  }
}
```

> **Hot reload:** OpenClaw snapshots the eligible skill list when a session starts. After adding or modifying skills, start a new session for changes to take effect (unless the skills watcher is enabled).

Once the skill is installed, proceed to [Step 2: Install lanbow-ads CLI](#step-2-install-lanbow-ads-cli).

---

## Platform 2: Claude Code

Claude Code loads skills from `~/.claude/skills/<skill-name>/`. Each skill directory must contain a `SKILL.md` as its entry point.

### Target Structure

```
~/.claude/skills/lanbow-ads-skills/
├── SKILL.md                              # Skill entry point (required)
└── references/
    ├── strategy-research.md
    ├── strategy-template.md
    ├── strategy-meta-only-template.md
    ├── strategy-document-standards.md
    ├── creative-generation.md
    ├── ad-delivery.md
    ├── ad-delivery-commands.md
    └── post-campaign-review.md
```

### Install

```bash
# 1. Clone the repo
git clone https://github.com/sandwichlab-ai/lanbow-claw-skill.git
cd lanbow-claw-skill

# 2. Copy skill files to the Claude Code skills directory
mkdir -p ~/.claude/skills/lanbow-ads-skills/references
cp lanbow-claw-skill/SKILL.md ~/.claude/skills/lanbow-ads-skills/
cp lanbow-claw-skill/references/*.md ~/.claude/skills/lanbow-ads-skills/references/

# 3. Verify
ls ~/.claude/skills/lanbow-ads-skills/
# Expected output: SKILL.md  references
```

> **Project-level install (optional):** To scope the skill to a single project, place the skill files under `.claude/skills/lanbow-ads-skills/` in your project root. The skill will only be active when Claude Code is launched from that project directory.

---

## Step 2: Install lanbow-ads CLI

`lanbow-ads` is the CLI tool for the Meta Ads API, required by Feature 3 (Ad Delivery) and Feature 4 (Post-Campaign Review).

```bash
npm install -g lanbow-ads
```

Verify the installation:

```bash
lanbow-ads --version
```

> **Prerequisites:** Requires Node.js >= 18 and npm. If not installed, use [nvm](https://github.com/nvm-sh/nvm) or [fnm](https://github.com/Schniz/fnm) to manage Node.js versions.

---

## Step 3: Configure Meta Ad Account

Feature 3 (Ad Delivery) and Feature 4 (Post-Campaign Review) require Meta ad account authorization.

> For a detailed step-by-step tutorial, see [meta-account-setup.md](lanbow-claw-skill/references/meta-account-setup.md)

### 3.1 Create a Meta App

1. Go to [Meta for Developers](https://developers.facebook.com/apps/)
2. **Create App** → Choose **Other** → Select **Business** type
3. Note your **App ID** and **App Secret**
4. Under **Add Products**, enable **Marketing API**
5. Under **Facebook Login > Settings**, add the redirect URI: `http://localhost:8080/callback`

### 3.2 Authenticate the CLI

```bash
# Save App credentials
lanbow-ads config set --app-id YOUR_APP_ID --app-secret YOUR_APP_SECRET

# OAuth login (opens browser automatically)
lanbow-ads auth login

# Set default ad account
lanbow-ads accounts list                        # List available accounts
lanbow-ads config set --account act_XXXXXXXXX   # Set default account

# Verify
lanbow-ads auth status
```

---

## Step 4: Configure Gemini API Key (Optional)

Feature 2 (Creative Generation) uses Google Gemini to generate ad images. This requires an API key.

1. Go to [Google AI Studio](https://aistudio.google.com/apikey) and create an API key
2. Set it as an environment variable:

```bash
# macOS / Linux — add to ~/.zshrc or ~/.bashrc to persist
export GEMINI_API_KEY="your-api-key"

# Windows PowerShell
$env:GEMINI_API_KEY = "your-api-key"
```

Models used:
- `gemini-2.5-flash` — creative proposal generation (text)
- `gemini-3.1-flash-image-preview` — ad image generation

---

## Feature Dependency Summary

| Feature | Description | Dependency | Required? |
|---------|-------------|------------|-----------|
| 1. Strategy Research | Market research & competitive analysis | WebSearch / WebFetch (built-in) | No extra setup |
| 2. Creative Generation | AI-powered ad image generation | `GEMINI_API_KEY` | Only if using this feature |
| 3. Ad Delivery | Campaign management | `lanbow-ads` CLI + Meta account | Only if using this feature |
| 4. Post-Campaign Review | Performance diagnosis & optimization | `lanbow-ads` CLI (same as Feature 3) | Only if using this feature |

> Each feature works independently — only install the dependencies you need.

---

## Verify Installation

```bash
# Skill files in place?
# OpenClaw (managed):
ls ~/.openclaw/skills/lanbow-ads-skills/SKILL.md
# OpenClaw (personal agent):
ls ~/.agents/skills/lanbow-ads-skills/SKILL.md
# OpenClaw (project-level):
ls skills/lanbow-ads-skills/SKILL.md
# Claude Code:
ls ~/.claude/skills/lanbow-ads-skills/SKILL.md
# CLI working?
lanbow-ads auth status          # Should show "Authenticated"
lanbow-ads campaigns list       # Should return a campaign list (or empty list, no errors)

# Gemini API working? (optional)
curl -s "https://generativelanguage.googleapis.com/v1beta/models?key=$GEMINI_API_KEY" | head -5
# Should return a JSON model list
```

---

## Uninstall

**OpenClaw:**

```bash
# Remove from whichever location you installed to:
rm -rf ~/.openclaw/skills/lanbow-ads-skills     # managed
rm -rf ~/.agents/skills/lanbow-ads-skills        # personal agent
rm -rf skills/lanbow-ads-skills                  # project-level
npm uninstall -g lanbow-ads
```

**Claude Code:**

```bash
rm -rf ~/.claude/skills/lanbow-ads-skills
npm uninstall -g lanbow-ads
```

---

## Troubleshooting

### `lanbow-ads: command not found`

```bash
# Confirm global installation
npm install -g lanbow-ads

# If npm global installs have permission issues, use nvm or fnm to manage Node.js (avoids sudo)
```

### `AuthRequiredError` / Token Expired

```bash
lanbow-ads auth login --force    # Re-authenticate
lanbow-ads auth status           # Check token status
```

### `Invalid App ID`

```bash
lanbow-ads config list           # Check stored app-id
lanbow-ads config set --app-id CORRECT_APP_ID
```

### Gemini API Returns 401 / 403

Your API key is invalid or expired. Generate a new one at [Google AI Studio](https://aistudio.google.com/apikey).

### Meta API Rate Limit

The Meta Marketing API has per-account rate limits. Wait a few minutes and retry, or reduce the `--limit` parameter on list commands.
