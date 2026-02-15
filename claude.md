# OpenClaw Setup Guide

## Repository Overview

**OpenClaw** is a self-hosted messaging gateway connecting platforms (WhatsApp, Telegram, Discord, iMessage) to AI coding agents.

### Repository Structure
```
openclawcourse/
├── index.md                   # Course navigation
├── 00-openclaw.md             # What is OpenClaw
├── 01-install-first-run.md    # Installation guide
├── 02-workspace-memory.md     # Workspace concepts
├── 03-pinchboard.md           # Pinchboard integration
├── 04-personal-assistant.md   # Assistant setup
├── 05-skills.md               # Skills system
├── 06-multi-agent.md          # Multi-agent routing
├── 07-security.md             # Security considerations
├── 08-sandboxing.md           # Docker isolation
```

### Requirements
- Node.js >= 22
- npm
- macOS or Linux

### Installation
```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

### Key Commands
```bash
openclaw status                    # Channel health
openclaw health                    # Gateway health
openclaw security audit --deep     # Security audit
openclaw doctor                    # Health checks
```

### Workspace Location
- `~/.openclaw/` - credentials, sessions, agent configs

---

## Security Isolation

### UTM Ubuntu VM on Mac
| Layer | Status |
|-------|--------|
| Filesystem | Isolated (no shared folders) |
| Network | Isolated (NAT mode) |
| Memory | Sandboxed |
| Processes | Cannot interact with macOS |

**Mac is safe** - VM cannot access Mac files unless shared folders are added.

### AWS EC2 Instance
| Risk Area | Mitigation |
|-----------|------------|
| Other EC2 instances | Isolated security group |
| AWS Account/IAM | Launch with **no IAM role** |
| Other AWS services | No IAM role = no access |
| VPC resources | Use isolated subnet |

**EC2 Security Checklist:**
- [ ] No IAM role attached
- [ ] Dedicated security group (SSH from your IP only)
- [ ] Isolated VPC/subnet
- [ ] No AWS credentials stored on instance

---

## EC2 Cost Estimation

### Minimum Setup (t3.small)
| Resource | Spec | Cost/month |
|----------|------|------------|
| Instance | 2 vCPU, 2GB RAM | $15.18 |
| Storage | 20GB gp3 | $1.60 |
| Transfer | 10GB out | $0.90 |
| **Total** | | **~$18** |

### Recommended Setup (t3.medium)
| Resource | Spec | Cost/month |
|----------|------|------------|
| Instance | 2 vCPU, 4GB RAM | $30.37 |
| Storage | 30GB gp3 | $2.40 |
| Transfer | 20GB out | $1.80 |
| **Total** | | **~$35** |

### Cost-Saving Options
| Option | Cost/month |
|--------|------------|
| Spot Instance (t3.small) | ~$5-6 |
| Reserved 1yr (t3.small) | ~$10 |
| Free Tier (t2.micro) | $0* |

*t2.micro (1GB RAM) may be tight but worth testing.

---

## Troubleshooting Steps Performed

### Upgrading Node.js (v20 -> v22)

OpenClaw requires Node.js >= 22. If installed via NodeSource apt repo:

```bash
# Remove old NodeSource repo
sudo rm /etc/apt/sources.list.d/nodesource.sources

# Install NodeSource setup for Node 22
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -

# Install/upgrade
sudo apt-get install -y nodejs

# Verify
node -v   # should show v22.x.x
```

Alternative (no sudo, using nvm):
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc
nvm install 22
nvm use 22
```

### Enabling Browser Control (Chrome)

OpenClaw's browser tool requires a Chromium-based browser (Chrome DevTools Protocol). Firefox is not supported.

**Important: Use Google Chrome, not snap Chromium.** The snap version of Chromium has internal sandboxing that prevents the OpenClaw gateway from launching it via CDP. This causes `Failed to start Chrome CDP on port 18800` errors even with `noSandbox: true`.

**Install Google Chrome on Ubuntu:**
```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt-get install -f -y
```

**Add browser config to `~/.openclaw/openclaw.json`:**

Add the following section at the top level of the JSON config:
```json
"browser": {
  "enabled": true,
  "executablePath": "/usr/bin/google-chrome-stable",
  "headless": true,
  "noSandbox": true,
  "defaultProfile": "openclaw"
}
```

| Config Key | Description |
|------------|-------------|
| `enabled` | Activate browser functionality |
| `executablePath` | Path to Chrome binary |
| `headless` | Run without GUI window |
| `noSandbox` | Disable Chrome's internal sandbox (needed for headless) |
| `defaultProfile` | Set `openclaw` as default (instead of `chrome` extension relay) |

**Restart the gateway and start the browser:**
```bash
openclaw gateway restart
sleep 5
openclaw browser start --browser-profile openclaw
openclaw browser status --browser-profile openclaw   # should show running: true
```

**Verify it's working:**
```
profile: openclaw
enabled: true
running: true
cdpPort: 18800
browser: custom
detectedPath: /usr/bin/google-chrome-stable
```

After this, the agent can use browser control for tasks like signing up on Pinchboard (`https://pinchboard.up.railway.app`).

### Switching from Gemini API Key to OAuth (Google Account Login)

Instead of using a Gemini API key, you can authenticate through your Google account (Gmail/Workspace) using the **Gemini CLI OAuth** plugin. This uses your existing Gemini subscription — no API billing.

**Step 1 — Install Gemini CLI:**
```bash
sudo npm install -g @google/gemini-cli
```

**Step 2 — Enable the plugin and restart gateway:**
```bash
openclaw plugins enable google-gemini-cli-auth
openclaw gateway restart
```

**Step 3 — Login via OAuth (opens browser for Google login):**
```bash
openclaw models auth login --provider google-gemini-cli --set-default
```

This opens a browser window. Log in with your Google account and grant access. The callback is captured automatically on `localhost:8085`.

**Step 3.5 — Enable Gemini 3 preview models (if needed):**

After installing Gemini CLI, run `gemini` in your terminal. Go to **Settings** and enable **preview features** to get access to Gemini 3 models. Without this, you may only see older models.

**Step 4 — Switch to the correct model:**

The OAuth login defaults to `gemini-3-pro-preview`, but this model may return a `429: No capacity available` error. Switch to `gemini-3-flash-preview` which has available capacity:

In `~/.openclaw/openclaw.json`, set:
```json
"agents": {
  "defaults": {
    "model": {
      "primary": "google-gemini-cli/gemini-3-flash-preview"
    },
    "models": {
      "google-gemini-cli/gemini-3-flash-preview": {}
    }
  }
}
```

Then restart:
```bash
openclaw gateway restart
```

**Step 5 — Verify:**
```bash
openclaw models status
```

Expected output:
```
Default       : google-gemini-cli/gemini-3-flash-preview
Configured models (1): google-gemini-cli/gemini-3-flash-preview

Auth overview
- google-gemini-cli effective=profiles | oauth=1
  - google-gemini-cli:user@example.com (OAuth) ok expires in ~54m

OAuth/token status
- google-gemini-cli usage: Pro 100% left · Flash 100% left
```

**Step 6 — Remove old API key config from `~/.openclaw/openclaw.json`:**

Remove the old `google:default` API key profile from `auth.profiles`:
```json
// REMOVE this:
"google:default": {
  "provider": "google",
  "mode": "api_key"
}
```

Remove the old model from `agents.defaults.models`:
```json
// REMOVE this:
"google/gemini-2.5-flash": {}
```

The final auth config should look like:
```json
"auth": {
  "profiles": {
    "google-gemini-cli:user@example.com": {
      "provider": "google-gemini-cli",
      "mode": "oauth"
    }
  }
},
"agents": {
  "defaults": {
    "model": {
      "primary": "google-gemini-cli/gemini-3-flash-preview"
    },
    "models": {
      "google-gemini-cli/gemini-3-flash-preview": {}
    }
  }
}
```

**Notes:**
- OAuth tokens auto-refresh — no need to log in again
- Quota is tied to your Google account's Gemini plan (Pro/Flash usage shown in `openclaw models status`)
- `gemini-3-pro-preview` may give 429 "No capacity available" errors — use `gemini-3-flash-preview` instead
- Alternative: use `google-antigravity-auth` plugin instead if you don't want to install Gemini CLI separately

---

## Audio Transcription (Whisper via Groq)

OpenClaw transcribes voice messages and audio files via a Python script that calls the Groq Whisper API (free tier, 164x real-time speed).

### Setup

**1. Install ffmpeg (required for audio format conversion):**
```bash
sudo apt install -y ffmpeg
```

**2. Get a free Groq API key** from https://console.groq.com

**3. Add Groq key to `~/.openclaw/openclaw.json` env.vars:**
```json
"GROQ_API_KEY": "gsk_your_key_here"
```

**4. Install groq Python package in shared venv:**
```bash
cd ~/.openclaw/workspace/skills/dalle-image-generator && uv add groq
```

**5. Restart:**
```bash
openclaw gateway restart
```

### How It Works

The agent uses the `transcribe.py` script via `exec`:

```
exec {command: '/home/hkord/.openclaw/workspace/skills/dalle-image-generator/.venv/bin/python /home/hkord/.openclaw/workspace/skills/transcribe-orchestrator/transcribe.py --file "<audio_file_path>"'}
```

- Auto-converts OGG/Opus (Telegram voice format) to MP3 via ffmpeg
- Auto-splits files over 25MB into chunks
- Supports 99+ languages with auto-detection
- Returns JSON with `transcript`, `duration_seconds`, `chunks_processed`

**Script location:** `skills/transcribe-orchestrator/transcribe.py`

### Cost

| Provider | Cost/Hour | Speed | Diarization |
|----------|----------|-------|-------------|
| Groq (free tier) | $0.00 | 164x real-time | No |
| OpenAI gpt-4o-transcribe | $0.36 | ~1x real-time | Yes |

---

## Orchestrator Skills

### task-orchestrator (Master)
The master orchestrator (`skills/task-orchestrator/`) provides the agent with a complete map of all available skills, native tools, and how to chain them together. It serves as the agent's reference for:
- Deciding which skills/tools to use for any given task
- Breaking complex tasks into granular steps
- Knowing all available Twitter, image, search, and audio capabilities
- Error handling and fallback strategies
- Delegating to specialized orchestrators for specific domains

### transcribe-orchestrator (Audio/Meetings)
A dedicated orchestrator (`skills/transcribe-orchestrator/`) for all audio transcription tasks:
- Voice message transcription (automatic on Telegram)
- Meeting transcription with structured summaries (topics, decisions, action items)
- Speaker diarization via OpenAI when needed
- Handles chunking for long files (>25MB)
- Provider selection (Groq free vs OpenAI paid)

### Current Skills Map

| Skill | Type | Purpose |
|-------|------|---------|
| `brave-search` | Instructions | Web search via Brave API |
| `dalle-image-generator` | Script | Generate images (gpt-image-1.5) |
| `image-prompt-generator` | Script | Generate DALL-E prompts from news via GPT-4o |
| `iran-hawks-engagement` | Instructions | Prepare responses to Iran hawks' tweets |
| `iran-awareness-influencers` | Instructions | Comment on global influencer tweets |
| `iran-revolution-engagement` | Instructions | Search and reply to Iran revolution tweets |
| `iran-revolution-media-generator` | Orchestrator | Full pipeline: research → image → tweet with media |
| `iran-protest-media-poster` | Orchestrator | Download real protest photos from web → tweet with media |
| `twitter-topic-engagement` | Orchestrator | Find tweets about a topic (Brave + timeline + key accounts) → reply |
| `tweet-crafter` | Guidelines | General-purpose tweet/reply text generator — no fixed templates, unique output each run |
| `twitter-image-fetcher` | Script | Fetch real photos from curated Iran-movement Twitter accounts (11 accounts, rotation, MEK filter) |
| `mek-content-filter` | Guardrail | Filters out MEK/NCRI/Maryam Rajavi content from all Iran pipelines |
| `task-orchestrator` | Orchestrator | Master task router and skill reference |
| `transcribe-orchestrator` | Orchestrator | Audio transcription and meeting notes |
| `twitter-follower-growth` | Instructions | Find and follow Iran revolution community accounts |
| `twitter-id-updater` | Script + Cron | Auto-update Twitter GraphQL query IDs |
| `mcporter` | Instructions | MCP CLI reference |

### Model Configuration

Primary: OpenAI GPT-4o-mini (paid, ~$0.15/$0.60 per million tokens). Gemini available as manual switch-back.

```json
"model": {
  "primary": "openai/gpt-4o-mini"
},
"models": {
  "google-gemini-cli/gemini-2.5-pro": {},
  "google-gemini-cli/gemini-2.5-flash": {},
  "google-gemini-cli/gemini-3-flash-preview": {},
  "google-gemini-cli/gemini-3-pro-preview": {},
  "openai/gpt-4o-mini": {}
}
```

**⚠️ Fallback does NOT work:** OpenClaw's config validator rejects the `"fallbacks"` key (`Unrecognized key: "fallback"` in gateway logs). The gateway does not auto-switch models on 429 errors. You must manually change `"primary"` in `openclaw.json` and restart the gateway.

**To switch back to Gemini** (when capacity is available):
```json
"primary": "google-gemini-cli/gemini-2.5-flash"
```

**Note:** Gemini OAuth free tier quota is shared across all models. Preview models (gemini-3-*) have very low limits (~250 RPD). `gemini-2.5-flash` has the highest free quota.

### Gemini 2.5 Flash vs GPT-4o-mini: Skill Execution Comparison

| Capability | Gemini 2.5 Flash | GPT-4o-mini |
|---|---|---|
| Read SKILL.md and follow workflow | Reliable | Unreliable — misinterprets skill name as creative task, uses wrong tools |
| Inline step-by-step in cron payload | Works | Works |
| Spawn sub-agents in isolated cron | Works (usually) | Fails (permission/access errors) |
| Context window | 1M tokens | 128K tokens |
| Cost | Free (OAuth quota) | ~$0.15/$0.60 per million tokens |
| Availability | Intermittent 429 "No capacity available" | Stable |

**Observed gpt-4o-mini failures:**
- "Run iran-protest-media-poster skill" → created a markdown file instead of executing the workflow
- "Read SKILL.md and follow steps" → used `twitter_post_tweet` (text-only) instead of `twitter_post_tweet_with_media`
- "Follow the twitter-topic-engagement skill" → tried to spawn sub-agents, got permission error

**Recommendation:** Always use inline step-by-step instructions for cron jobs regardless of model. It works with both, and you won't need to rewrite payloads when switching. SKILL.md files are still useful as documentation and for on-demand interactive runs (especially with Gemini), but for unattended cron execution, inline is the reliable path.

---

## MEK Content Filter Guardrail

MEK (Mojahedin-e Khalq) is a cult-like organization NOT supported by the Iranian people. Their content must be excluded from all Iran-related pipelines.

**Skill location:** `skills/mek-content-filter/`

**What it blocks:**
- **Domains:** iranfreedom.org, mek-iran.com, ncr-iran.org, mojahedin.org, maryam-rajavi.com, iranfocus.com, camp-ashraf.com + partial matches
- **Strong keywords** (single match = block): Maryam Rajavi, Masoud Rajavi, Mojahedin-e Khalq, PMOI/MEK, Camp Ashraf, NCRI president, etc.
- **Medium keywords** (2+ matches = block): rajavi, mek, ncri, pmoi, mojahedin, etc.

**Usage as CLI:**
```bash
python mek_filter.py --url "https://iranfreedom.org/photo.jpg" --text "Some article title"
# Returns: {"blocked": true, "reason": "URL domain matches MEK blocklist: iranfreedom.org"}
```

**Integration:** Built into `download_image.py` (iran-protest-media-poster) at three levels — after image search, after web search fallback, and before download. Also added as agent-level guardrail instructions in `iran-revolution-media-generator`, `twitter-topic-engagement`, `iran-hawks-engagement`, and `iran-awareness-influencers` SKILL.md files.

---

## Twitter GraphQL Query ID Auto-Updater

Twitter rotates their internal GraphQL query IDs every 2-4 weeks, causing 404 errors on search/timeline tools.

**Automated fix:** A script fetches latest IDs from [fa0311/twitter-openapi](https://github.com/fa0311/twitter-openapi) (auto-updated daily), updates the source, and rebuilds.

**Manual run:**
```bash
bash ~/.openclaw/workspace/skills/twitter-id-updater/update_query_ids.sh
openclaw gateway restart
```

**Cron:** Set up on OpenClaw to run every 2 weeks. Also triggers on any Twitter 404 error.

**Skill location:** `skills/twitter-id-updater/`

### SearchTimeline Account-Level Block

The `SearchTimeline` GraphQL endpoint returns 404 for this Twitter account regardless of query IDs, features, or request method (raw HTTP, browser-context fetch, Puppeteer). This is an account-level restriction — Twitter blocks search API access for accounts flagged as automated.

**Workaround implemented in `twitter-client.ts`:**
- `searchTweets("from:handle")` automatically falls back to `getUserTweets(handle)` — no browser launch needed
- `searchTweets("@handle ...")` tries SearchTimeline first, on 404 falls back to `getUserTweets` for the first mentioned handle
- All skills and cron jobs updated to use `twitter_get_user_tweets` instead of `twitter_search_tweets`

**Working endpoints:** `UserByScreenName`, `UserTweets`, `TweetResultByRestId`, `HomeTimeline`, `CreateTweet` (via Puppeteer), `Followers`, `Following`
**Broken endpoint:** `SearchTimeline` (404, account-level block)

### Per-Endpoint Feature Flags

Twitter requires DIFFERENT feature flag objects per GraphQL endpoint. Sending the wrong features causes 400 errors. The `twitter-client.ts` now has three separate feature objects:

| Feature Set | Used By |
|-------------|---------|
| `TIMELINE_FEATURES` (31 flags) | SearchTimeline, UserTweets, HomeTimeline, TweetDetail |
| `USER_PROFILE_FEATURES` (12 flags) | UserByScreenName |
| `TWEET_RESULT_FEATURES` (25 flags) | TweetResultByRestId |

### Reply Reliability (`twitter_reply_to_tweet`)

The Puppeteer-based reply tool had intermittent "Waiting for selector" failures due to `networkidle2` hanging on Twitter's persistent connections. Fixed by switching to `domcontentloaded` + 3s render delay + 3-attempt retry (15s timeout each). All skills now use `twitter_get_user_tweets` (not search) and `twitter_reply_to_tweet` with proper MCP tool names.

### Media Upload Verification (`twitter_post_tweet_with_media`)

The Puppeteer-based media posting tool had a silent failure mode: after calling `.uploadFile()`, it waited a fixed 5 seconds without verifying the image actually attached to the DOM. If the upload failed (Twitter rejection, network error, wrong format), the tweet posted as **text-only** and the tool returned `"Tweet with media posted successfully"` — a false positive.

**Fix:** Replaced blind 5-second wait with a DOM verification loop:
- Checks every 1 second (up to 10 seconds) for image thumbnail selectors in the compose UI
- Selectors checked: `[data-testid="attachments"] img`, `img[src*="blob:"]`, `[role="progressbar"]`
- If thumbnail found → proceeds with 2-second processing buffer
- If no thumbnail after 10 seconds → **throws error**, tweet is NOT posted
- Prevents text-only tweets from being posted when image upload fails

### Repo Consolidation

Two Twitter MCP repos existed: `agent-twitter-mcp` (inactive, legacy) and `twitter-mcp-standalone` (active). Ported three useful features from `agent-twitter-mcp` → `twitter-mcp-standalone`, then deleted `agent-twitter-mcp`.

**New tools added:**
| Tool | Type | Description |
|------|------|-------------|
| `twitter_get_followers` | HTTP read | Get a user's followers list |
| `twitter_get_following` | HTTP read | Get accounts a user is following |
| `twitter_quote_tweet` | Puppeteer write | Quote tweet with commentary text |

**Active repo:** `~/coding/my_github/twitter-mcp-standalone/` (only one now)

---

## Cron Jobs: How They Work and Limitations

### Architecture

OpenClaw cron has two layers:

1. **`cron` tool (agent-side)** — The agent calls `cron.list()`, `cron.create()`, `cron.delete()` via an API to the gateway process. This operates within the agent's current session context.

2. **`jobs.json` (disk-side)** — The persistent storage file at `~/.openclaw/cron/jobs.json`. The gateway loads this on startup and writes back to it when job state changes (e.g., after a run completes).

### Known Limitation: Agent Can't See/Manage Existing Cron Jobs

Jobs created in a previous agent session (or directly in `jobs.json`) are **not visible** to the agent's `cron.list()` tool in a new conversation. This is because:
- Each cron run uses `"sessionTarget": "isolated"` — every execution spins up a fresh, isolated session
- The `cron` tool scopes by session context and doesn't expose jobs it didn't create
- The agent will report "I can't find that cron job" even though it exists and is running

### How to Manage Cron Jobs Manually

Since the agent can't manage jobs via `cron.list()`/`cron.delete()`, edit `jobs.json` directly:

**Disable a job:**
```json
"enabled": false
```

**Then restart the gateway** so it reloads from disk:
```bash
openclaw gateway restart
```

**Important:** The gateway keeps jobs in memory and periodically writes them back to `jobs.json`, which can overwrite manual edits. Always restart the gateway **immediately** after editing `jobs.json` so it loads the new version before it overwrites your changes.

### Current Cron Jobs

| Job | Schedule | Status | Payload |
|-----|----------|--------|---------|
| Iran Hawks Engagement Loop | Every 2 hours | Enabled | Uses `get_user_tweets` (not search) + `iran-hawks-engagement` skill + `facts.md` |
| Twitter Follower Growth v2 | Every 1 hour | **Disabled** | References `twitter-follower-growth` skill |
| Iran Revolution Media Generator | Every 3 hours | Enabled | Inline steps: news search → DALL-E image (120s timeout) → tweet-crafter text → post with media + verification |
| Iranian Global March Engagement | Every 1 hour | Enabled | Inline steps with `tweet-crafter` rules for context-aware replies (no fixed template) |
| Iran Protest Media Poster | Every 1 hour | Enabled | Inline steps with `tweet-crafter` rules for unique city-specific tweets, post verification via `get_user_tweets` |

**File:** `~/.openclaw/cron/jobs.json`

### Cron Payload Best Practice: Inline Step-by-Step Instructions

When writing cron job payloads, **use fully inline step-by-step commands** instead of referencing SKILL.md files. Smaller models (gpt-4o-mini) don't reliably read and follow complex skill files — they may interpret the skill name as a creative task instead of executing the workflow.

**Bad (unreliable):**
```
"message": "Run the iran-protest-media-poster skill. Read the skill at skills/iran-protest-media-poster/SKILL.md and execute the full workflow."
```

**Good (reliable):**
```
"message": "Execute these exact steps in order. You MUST complete ALL steps.\n\nStep 1: Read memory/state.json to get last_city_index.\n\nStep 2: Pick the NEXT city from this list: [Sydney, Melbourne, Munich]\n\nStep 3 — DOWNLOAD IMAGE (MANDATORY): Run this exact exec command:\nexec {command: 'python /path/to/download_image.py --query \"CITY query\" --city \"CITY\"'}\n\nStep 4: Craft tweet text.\n\nStep 5 — POST WITH IMAGE (MANDATORY): Use twitter_post_tweet_with_media (NOT twitter_post_tweet).\n\nStep 6: Update state file.\n\nStep 7: Report results."
```

**Key rules for inline payloads:**
- Number every step explicitly (Step 1, Step 2, etc.)
- Mark critical steps as **(MANDATORY)** in bold
- Include exact tool names and exec commands — don't assume the agent knows them
- Add negative instructions for common mistakes ("Do NOT use twitter_post_tweet — that posts text only")
- Keep state management simple (JSON file with index rotation)
- Include error handling instructions per step ("If error, try next city")
