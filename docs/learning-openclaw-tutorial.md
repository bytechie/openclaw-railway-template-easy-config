# Learning Openclaw Tutorial

> A comprehensive guide to deploying, configuring, and integrating Openclaw with Atlas Cloud and messaging platforms.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Overview](#overview)
3. [Quick Start](#quick-start)
4. [Deployment](#deployment)
5. [Configuration](#configuration)
6. [Integrations](#integrations)
7. [Use Case Topics](#use-case-topics)
8. [Security Hardening](#security-hardening)
9. [Testing & Validation](#testing--validation)
10. [Troubleshooting](#troubleshooting)
11. [References](#references)

---

## Prerequisites

Before starting this tutorial, ensure you have:

### Required Accounts

| Service | Purpose | Cost |
|---------|---------|------|
| [Railway](https://railway.app/) | Hosting platform | Free tier available, ~$5/month for paid |
| [Atlas Cloud](https://www.atlascloud.ai/) | AI model provider | Free tier + pay-per-use |
| [GitHub](https://github.com/) | Code hosting | Free |

### Required Tools

- **Git** - For cloning the repository
- **Docker** (optional) - For local testing
- **Basic CLI knowledge** - For running commands

### Skill Level

- **Beginner to Intermediate** - No prior Openclaw experience required
- **Basic understanding** of environment variables and API keys

### Time Estimate

- **Quick Start**: 10 minutes
- **Full Tutorial**: 1-2 hours
- **Each Integration**: 15-30 minutes

---

## Overview

### What is Openclaw?

Openclaw (formerly Moltbot/Clawdbot) is an open-source AI coding assistant platform that:

- **Provides AI-powered code generation** - Generate, refactor, and debug code
- **Integrates with messaging platforms** - Discord, Telegram, Slack, LINE, etc.
- **Supports multiple AI providers** - Anthropic, OpenAI, Google, Atlas Cloud, and more
- **Offers extensible architecture** - Skills, plugins, custom agents
- **Runs self-hosted** - Your data stays on your servers

### Key Architecture Components

```
┌─────────────────────────────────────────────────────────────┐
│                    Railway Deployment                        │
│                      *.up.railway.app                        │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Express Wrapper (src/server.js)                 │
│                   Port: 8080 (external)                     │
│  ┌──────────────────────────────────────────────────────┐    │
│  │ /setup (Basic Auth) → Setup Wizard                   │    │
│  │ /openclaw, /* → Proxy to Gateway (Bearer Token)     │    │
│  └──────────────────────────────────────────────────────┘    │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│               Openclaw Gateway (localhost:18789)            │
│   ┌────────────────┐    ┌────────────────┐                  │
│   │  Agent Engine  │    │  Auth Store    │                  │
│   │  - AI Models   │    │  - API Keys     │                  │
│   │  - Skills      │    │  - Profiles     │                  │
│   └────────────────┘    └────────────────┘                  │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                  AI Providers                               │
│   ┌──────────┐ ┌─────────┐ ┌──────────┐ ┌─────────┐     │
│   │Atlas Cloud│ │Anthropic│ │ OpenAI   │ │ Google  │     │
│   └──────────┘ └─────────┘ └──────────┘ └─────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### Common Use Cases

| Use Case | Difficulty | Description |
|----------|-----------|-------------|
| **24/7 Personal Assistant** | Easy | AI chatbot for general questions and tasks |
| **Code Review & Debugging** | Medium | Analyze code, find bugs, suggest fixes |
| **Image/Video Generation** | Medium | Generate visual content using Atlas Cloud models |
| **Financial Analysis** | Advanced | Process financial data, generate reports |
| **Security Auditing** | Advanced | Scan code for vulnerabilities |
| **Social Media Management** | Easy | Schedule posts, analyze engagement |

---

## Quick Start

Get Openclaw running on Railway in **5 minutes**:

### Step 1: Deploy to Railway (2 minutes)

<details>
<summary>Click to expand</summary>

1. Go to [railway.app](https://railway.app/) and log in
2. Click **New Project** → **Deploy from GitHub repo**
3. Enter: `https://github.com/[your-username]/openclaw-railway-template-easy-config`
4. **Add a Volume** mounted at `/data`
5. Set environment variable: `SETUP_PASSWORD=your-secure-password`
6. Click **Deploy**

Wait for deployment to complete (~2 minutes).
</details>

### Step 2: Run Setup Wizard (3 minutes)

<details>
<summary>Click to expand</summary>

1. Visit `https://your-app.up.railway.app/setup`
2. Login with your `SETUP_PASSWORD`
3. Select **Atlas Cloud** as provider
4. Paste your Atlas Cloud API key
5. Click **Run Setup**

That's it! Your Openclaw instance is ready.
</details>

### Success Indicators

✅ Setup completes without errors
✅ Gateway starts: `[gateway] ready at /openclaw`
✅ Control UI accessible at `/openclaw`

---

## Deployment

### Deployment Processes to Railway Platform

**Learning Objectives:**
- ✓ Create a Railway project from this template
- ✓ Configure persistent storage
- ✓ Set up environment variables
- ✓ Verify deployment health

**Time:** 15 minutes

### Option 1: Deploy from GitHub Template (Recommended)

<details>
<summary>Step-by-step instructions</summary>

1. **Prepare your GitHub repository**
   ```bash
   # Fork this repository to your GitHub account
   # Or push your own version
   ```

2. **Create Railway Project**
   - Go to [railway.app](https://railway.app/)
   - Click **New Project** → **Deploy from GitHub repo**
   - Select your forked repository
   - Click **Import**

3. **Configure the Service**
   - **Root Directory**: `/`
   - **Builder**: `Dockerfile`
   - **Add Variables**:
     - `SETUP_PASSWORD` - Your secure password

4. **Add Persistent Volume**
   - Click **Settings** → **Variables**
   - Click **New Volume**
   - Mount path: `/data`
   - Name: `openclaw-data`

5. **Enable Public Networking**
   - Click **Settings** → **Networking**
   - Enable **Public Networking**
   - Railway will assign a domain like `https://your-app.up.railway.app`

6. **Deploy**
   - Click **Deploy**
   - Wait for build and deployment (~5 minutes)

7. **Verify Health**
   - Visit `https://your-app.up.railway.app/setup/healthz`
   - Should return: `{"ok":true}`
</details>

### Option 2: Manual Deployment from Local

<details>
<summary>For custom builds or development</summary>

```bash
# Clone the repository
git clone https://github.com/[your-username]/openclaw-railway-template-easy-config.git
cd openclaw-railway-template-easy-config

# Build the Docker image
docker build -t openclaw-railway-template .

# Run locally (for testing)
docker run --rm -p 8080:8080 \
  -e PORT=8080 \
  -e SETUP_PASSWORD=test \
  -v $(pwd)/.tmpdata:/data \
  openclaw-railway-template

# Or push to your own registry
docker tag openclaw-railway-template ghcr.io/[your-username]/openclaw:latest
docker push ghcr.io/[your-username]/openclaw:latest
```
</details>

### Deployment Checklist

- [ ] Repository cloned or forked
- [ ] Railway project created
- [ ] Volume mounted at `/data`
- [ ] `SETUP_PASSWORD` environment variable set
- [ ] Public networking enabled
- [ ] Deployment successful
- [ ] Health check endpoint returns `{"ok":true}`

---

## Configuration

### General Tutorial

**Learning Objectives:**
- ✓ Navigate the setup wizard
- ✓ Configure AI providers
- ✓ Set up messaging channels
- ✓ Understand configuration files

**Time:** 20 minutes

### Setup Wizard Walkthrough

The setup wizard at `/setup` guides you through 5 steps:

#### Step 1: Welcome
Overview of Openclaw features and what you'll need.

#### Step 2: Authentication
Select your AI provider and enter your API key.

**Available Providers:**
- **Anthropic** (Claude) - Recommended for coding
- **OpenAI** (GPT models)
- **Google** (Gemini)
- **Atlas Cloud** - Multi-model with competitive pricing
- **OpenRouter** - Access to multiple models
- **Vercel AI Gateway**
- **Moonshot AI**, **Z.AI**, **MiniMax**, **Qwen**, **Synthetic**, **OpenCode Zen**

> ⚠️ **Common Mistake**: Entering an invalid API key
>
> **Solution**: Verify your API key by testing it directly:
> ```bash
> # Atlas Cloud
> curl -H "Authorization: Bearer YOUR_KEY" https://api.atlascloud.ai/v1/models
> ```

#### Step 3: Channels (Optional)
Configure messaging platforms:
- **Telegram** - Bot token from @BotFather
- **Discord** - Bot token from Discord Developer Portal
- **Slack** - Bot token and App token

> ⚠️ **Discord Requirement**: Enable **MESSAGE CONTENT INTENT** in your Discord bot settings or messages won't be received.

#### Step 4: Review
Verify your settings before running setup.

#### Step 5: Complete
Watch the setup progress and access your Openclaw instance.

### Configuration File Structure

```
/data/.openclaw/
├── openclaw.json          # Main configuration file
├── gateway.token           # Gateway authentication token
├── agents/
│   └── main/
│       └── agent/
│           └── auth-profiles.json  # AI provider API keys
└── channels/
    ├── telegram/
    ├── discord/
    └── slack/
```

---

## Integrations

### Integrating Openclaw with Atlas Cloud Platform

**Learning Objectives:**
- ✓ Understand Atlas Cloud → Venice provider mapping
- ✓ Configure API keys correctly
- ✓ Verify the integration works
- ✓ Compare with/without Atlas Cloud

**Time:** 15 minutes

#### What is Atlas Cloud?

Atlas Cloud is a GPU cloud platform providing:
- **100+ AI models** including LLMs, image, and video generation
- **OpenAI-compatible API** for easy integration
- **Competitive pricing** - Pay only for what you use
- **High-performance GPUs** for faster inference

#### How the Integration Works

Openclaw uses the internal provider name **"Venice"** for Atlas Cloud:

```
Setup Wizard  →  Atlas Cloud Selection
                  ↓
Wrapper Code  →  Maps to --venice-api-key flag
                  ↓
Openclaw CLI  →  Uses Venice provider
                  ↓
Auth Profile  →  { "default": { "provider": "venice", "apiKey": "..." } }
```

#### Atlas Cloud vs Other Providers

| Feature | Atlas Cloud | Anthropic | OpenAI |
|---------|-------------|-----------|--------|
| **Setup Complexity** | Easy | Easy | Easy |
| **Model Variety** | 100+ models | Claude models | GPT models |
| **Pricing** | Pay-per-use | Token-based | Token-based |
| **Special Features** | Image/Video gen | Extended thinking | Function calling |
| **Best For** | Multi-model projects | Coding tasks | General purpose |

#### Atlas Cloud Use Cases

**1. Image Generation**
```javascript
// Use Atlas Cloud for DALL-E-like image generation
const image = await generateImage({
  provider: "atlas",
  model: "stability-ai/sdxl-turbo",
  prompt: "A cat wearing sunglasses"
});
```

**2. Video Generation**
```javascript
// Generate short video clips
const video = await generateVideo({
  provider: "atlas",
  model: "zeroscope/luma-dream-machine",
  prompt: "A timelapse of a sunset"
});
```

**3. Cost-Effective Coding**
```javascript
// Use Atlas Cloud for coding tasks with alternative models
const code = await generateCode({
  provider: "atlas",
  model: "venice/zai-org/glm-4.7",
  prompt: "Write a Python function to sort a list"
});
```

#### Configuration Steps

<details>
<summary>Click to see detailed configuration</summary>

1. **Get Atlas Cloud API Key**
   - Visit [atlascloud.ai](https://www.atlascloud.ai/)
   - Sign up/login
   - Navigate to API Keys section
   - Create a new API key

2. **Select Atlas Cloud in Setup Wizard**
   - Choose "Atlas Cloud" from provider dropdown
   - Choose "Atlas Cloud API key" as auth method
   - Paste your API key

3. **Automatic Configuration**
   The wrapper automatically configures:
   ```javascript
   {
     "env.ATLAS_API_BASE": "https://api.atlascloud.ai/v1/",
     "env.ATLAS_API_MODEL": "zai-org/glm-4.7",
     "agents.main.model": "venice/zai-org/glm-4.7",
     "providers.venice.apiKey": "<your-key>"
   }
   ```

4. **Verify Integration**
   - Visit `/openclaw`
   - Send a test message
   - Check logs for: `[atlas] configured Atlas Cloud`
</details>

#### Comparing: With vs Without Atlas Cloud

| Aspect | Without Atlas Cloud | With Atlas Cloud |
|--------|---------------------|------------------|
| **Available Models** | Limited to configured provider | 100+ models available |
| **Cost** | Provider-dependent | Potentially lower costs |
| **Setup** | Simple API key entry | Simple API key entry |
| **Model Switching** | Requires reconfiguration | Easy model switching |

> ⚠️ **Common Pitfall**: Default model still shows as anthropic
>
> **Solution**: After setup, verify the agent model is set:
> ```bash
> openclaw config get agents.main.model
> # Should show: venice/zai-org/glm-4.7
> ```

---

### Integrating Openclaw with Discord

**Learning Objectives:**
- ✓ Create Discord application and bot
- ✓ Configure MESSAGE CONTENT INTENT
- ✓ Generate bot token and invite link
- ✓ Test Discord integration

**Time:** 20 minutes

#### Discord Integration Overview

```
User → Discord Server → Discord Bot → Openclaw Gateway → AI Response
```

#### Step-by-Step Setup

<details>
<summary>Click to expand Discord setup</summary>

1. **Create Discord Application**
   - Go to [Discord Developer Portal](https://discord.com/developers/applications)
   - Click **New Application**
   - Name it "Openclaw" or similar
   - Click **Create**

2. **Create Bot User**
   - Open the **Bot** tab
   - Click **Add Bot**
   - Copy the **Bot Token** (save this!)

3. **Enable Privileged Gateway Intents** ⚠️ IMPORTANT
   - In the Bot section, scroll to **Privileged Gateway Intents**
   - Enable:
     - ✅ **MESSAGE CONTENT INTENT** (Critical!)
     - ✅ Server Members Intent
   - Click **Save Changes**

4. **Configure OAuth2 Permissions**
   - Open **OAuth2** → **URL Generator**
   - Scopes: `bot`, `applications.commands`
   - Bot Permissions:
     - Send Messages
     - Read Messages/View Channels
     - Read Message History
     - Add Reactions
   - Copy generated URL and open in browser
   - Authorize the bot for your server

5. **Configure in Openclaw**
   - In setup wizard, go to Step 3 (Channels)
   - Expand **Discord Bot** section
   - Paste the bot token
   - Click **Run Setup**
</details>

#### Discord Use Cases

| Use Case | Description |
|----------|-------------|
| **Server Assistant** | Answer questions in Discord servers |
| **Code Help** | Debug code snippets shared in Discord |
| **Image Generation** | Generate images in channels with Atlas Cloud |
| **Admin Commands** | `/help`, `/status`, `/reset` commands |

> ⚠️ **Common Mistake**: Forgetting MESSAGE CONTENT INTENT
>
> **Solution**: Always enable this intent or the bot won't receive message content!

---

### Integrating Openclaw with Telegram

**Learning Objectives:**
- ✓ Create Telegram bot via BotFather
- ✓ Configure bot permissions
- ✓ Test Telegram integration

**Time:** 15 minutes

#### Step-by-Step Setup

<details>
<summary>Click to expand Telegram setup</summary>

1. **Create Telegram Bot**
   - Open Telegram and search for **@BotFather**
   - Send `/newbot`
   - Follow prompts to name your bot
   - BotFather will give you a token like: `123456789:ABC...`

2. **Configure Bot Permissions**
   - With BotFather, use `/setprivacy` → **Disabled**
   - This allows the bot to read all messages

3. **Configure in Openclaw**
   - In setup wizard, go to Step 3 (Channels)
   - Expand **Telegram Bot** section
   - Paste the bot token
   - Click **Run Setup**
</details>

#### Telegram Use Cases

| Use Case | Description |
|----------|-------------|
| **Personal Assistant** | DM the bot for private conversations |
| **Group Chats** | Add bot to groups for collaborative assistance |
| **File Sharing** | Share code files for analysis |
| **Inline Queries** | Use `@botname query` inline in chats |

> ⚠️ **Common Mistake**: Bot privacy settings enabled
>
> **Solution**: Use `/setprivacy` → **Disabled** in BotFather

---

## Use Case Topics

### 24/7 Personal Assistance

**Learning Objectives:**
- ✓ Set up always-available AI assistant
- ✓ Configure for different time zones
- ✓ Handle multiple concurrent users

**Configuration:**
```javascript
// In openclaw.json
{
  "gateway": {
    "mode": "local",
    "port": 18789
  },
  "agents": {
    "main": {
      "model": "venice/zai-org/glm-4.7"
    }
  }
}
```

**Best Practices:**
- Use Railway's always-free tier for testing
- Configure auto-restart on failure
- Set up monitoring for gateway health

### Integrating Skills

Openclaw supports extending functionality with **Skills**:

**Available Skills:**
- `banana` - Image understanding and analysis
- `github` - GitHub repository operations
- `notion` - Notion database integration
- `browser` - Web browsing and scraping

**Installing Skills:**
```bash
# In Railway console or SSH
openclaw skills install @openclaw/skill-banana
openclaw skills list
```

### Searching

Openclaw has built-in web search capabilities:

**Enable Web Search:**
```bash
# Set Brave API key
openclaw config set tools.web.braveApiKey YOUR_KEY

# Or use in setup wizard
```

**Usage:**
```
User: Search for the latest Openclaw release
Openclaw: [Searching...] Found version 2026.1.30
```

### Imaging and Video Generation

With Atlas Cloud integration:

**Image Generation:**
```javascript
// Available models:
- stability-ai/sdxl-turbo
- flux.1-dev
- midjourney-like models
```

**Video Generation:**
```javascript
// Available models:
- zeroscope/luma-dream-machine
- stable-video-diffusion
```

**Configuration:**
```bash
# Set image model
openclaw config set agents.main.model "venice/flux.1-dev"

# Generate image in chat
User: Generate an image of a sunset over mountains
```

### Financial Use Cases

**Applications:**
- Expense tracking and categorization
- Invoice processing (OCR + analysis)
- Financial report generation
- Market trend analysis

**Privacy Considerations:**
- Never share real financial data in public channels
- Use private DMs or self-hosted instances
- Redact sensitive numbers from logs

### Security Use Cases

**Applications:**
- Code vulnerability scanning
- Security audit automation
- Threat detection in logs
- Security policy documentation

**Best Practices:**
```javascript
// Enable security auditing
openclaw config set tools.exec.securityMode "ask"
openclaw config set tools.exec.safeBins ["/usr/bin/git"]

// Run security audit
openclaw security audit
```

### Privacy Best Practices

**Data Protection:**
1. **Use persistent volumes** - Data survives redeploys
2. **Set strong SETUP_PASSWORD** - Prevents unauthorized access
3. **Enable gateway authentication** - Token-based auth required
4. **Regular backups** - Export `/setup/export` regularly

**Log Redaction:**
```javascript
// Openclaw automatically redacts:
- API keys (partial)
- Passwords
- Sensitive tokens

// Verify:
openclaw config get logging.redaction
```

### Social Media Use Cases

**Platform-Specific Features:**

| Platform | Features |
|----------|----------|
| **Discord** | Slash commands, rich embeds, thread support |
| **Telegram** | Inline queries, file sharing, group chats |
| **Slack** | Workflow builder, slash commands, file uploads |

**Automation Examples:**
- Auto-moderation (content filtering)
- Scheduled posts
- Analytics reporting
- Community management

---

## Security Hardening

### How to Secure Login Console

**Current Implementation:**

The `/setup` wizard is protected by **HTTP Basic Authentication**:

```javascript
// src/server.js
function requireSetupAuth(req, res, next) {
  const header = req.headers.authorization || "";
  const [scheme, encoded] = header.split(" ");

  if (scheme !== "Basic" || !encoded) {
    res.set("WWW-Authenticate", 'Basic realm="Openclaw Setup"');
    return res.status(401).send("Auth required");
  }

  const decoded = Buffer.from(encoded, "base64").toString("utf8");
  const password = decoded.slice(decoded.indexOf(":") + 1);

  if (password !== SETUP_PASSWORD) {
    return res.status(401).send("Invalid password");
  }

  return next();
}
```

**Best Practices:**

1. **Use Strong Passwords**
   ```
   ❌ "password123"
   ✅ "P@ssw0rd!2026$Openclaw"
   ✅ Use Railway's ${{ secret() }} for random generation
   ```

2. **Enable HTTPS Only**
   - Railway automatically provides HTTPS
   - Redirect all HTTP traffic to HTTPS

3. **Change Password Regularly**
   - Rotate `SETUP_PASSWORD` periodically
   - Update in Railway Variables → Redeploy

### How to Redact Sensitive Information from Logs

**Openclaw's Built-in Redaction:**

Openclaw automatically redacts:
- API keys (shows first 4 and last 4 characters only)
- Passwords (fully redacted)
- Authentication tokens (partially redacted)

**Example Log Output:**
```
[onboard] Wrapper token: a1b2c3d4...5678 (length: 32)
[onboard] Config token:  a1b2c3d4...5678 (length: 32)
```

**Manual Log Redaction:**

For custom logging, use Openclaw's redaction utilities:

```javascript
import { redactApiKey } from '@openclaw/shared';

console.log(`Using API key: ${redactApiKey(apiKey)}`);
// Output: Using API key: sk-ant...xyz
```

**Environment Variable Redaction:**

```bash
# Don't log full API keys
echo "API_KEY configured"  # ✅ Good
echo "API_KEY=${API_KEY}"    # ❌ Bad - logs full key

# Use partial logging
echo "API_KEY=${API_KEY:0:8}..."  # ✅ Better - shows first 8 chars
```

### Security Checklist

- [ ] Strong `SETUP_PASSWORD` configured
- [ ] Gateway token authentication enabled
- [ ] HTTPS enforced (automatic on Railway)
- [ ] No plaintext credentials in logs
- [ ] Volume mounted at `/data` for persistence
- [ ] Firewall rules configured (if applicable)
- [ ] Regular security updates (`git pull` latest changes)

---

## Testing & Validation

### Testing Openclaw Installation

**Health Check Endpoint:**

```bash
curl https://your-app.up.railway.app/setup/healthz
# Should return: {"ok":true}
```

**Gateway Status:**

```bash
# SSH into Railway container
railway shell

# Check gateway status
openclaw status

# Check configuration
openclaw config get gateway
```

**Chat Test:**

1. Visit `/openclaw` (Control UI)
2. Send: "Hello, can you hear me?"
3. Verify response comes from configured model

### Testing Atlas Cloud Integration

**Step 1: Verify API Key**

```bash
curl -X GET "https://api.atlascloud.ai/v1/models" \
  -H "Authorization: Bearer YOUR_ATLAS_API_KEY"
```

**Step 2: Check Configuration**

```bash
# SSH into Railway container
railway shell

# Check agent model
openclaw config get agents.main.model
# Should return: "venice/zai-org/glm-4.7"

# Check auth profiles
cat /data/.openclaw/agents/main/agent/auth-profiles.json
# Should contain: {"default":{"provider":"venice","apiKey":"sk-..."}}
```

**Step 3: Test Chat**

```
User: What model are you using?
Openclaw: I'm using the GLM-4.7 model from Z.AI through Atlas Cloud.
```

### Testing Discord Integration

**Verification Steps:**

1. **Check Bot is Online**
   - Discord bot should show "Online" status

2. **Send Test Message**
   ```
   /help
   ```
   - Should display available commands

3. **Test in DM**
   - Direct message the bot
   - Should respond to messages

4. **Test in Group**
   - Add bot to a group
   - Mention the bot: `@botname help`

> ⚠️ **Common Issue**: Bot not responding
>
> **Check**:
> - MESSAGE CONTENT INTENT enabled?
> - Bot invited to server?
> - Token correctly pasted?

### Testing Telegram Integration

**Verification Steps:**

1. **Find Bot on Telegram**
   - Search for your bot username
   - Start conversation

2. **Test Commands**
   ```
   /start
   /help
   ```

3. **Test Group Chat**
   - Add bot to a group
   - Send: `/help` or mention bot

---

## Troubleshooting

### Common Deployment Issues

| Issue | Solution |
|-------|----------|
| **Build fails** | Check Dockerfile syntax, verify build logs |
| **Container crashes** | Check Railway logs for error messages |
| **Volume not mounted** | Verify Volume is configured at `/data` |
| **SETUP_PASSWORD not working** | Clear browser cache, verify variable is set |

### Atlas Cloud Integration Issues

**Issue:** "No API key found for provider 'anthropic'"

**Cause:** Agent model not configured to use Venice provider

**Solution:**
```bash
# SSH into container
railway shell

# Set agent model
openclaw config set agents.main.model "venice/zai-org/glm-4.7"

# Restart gateway
openclaw gateway restart
```

**Issue:** "unknown option '--venice-api-key'"

**Cause:** Using old version of Openclaw (pre-2026.1.29)

**Solution:**
- Ensure template builds from `main` branch
- Check Dockerfile `ARG OPENCLAW_GIT_REF=main`

**Issue:** API requests to Atlas Cloud failing

**Troubleshooting:**
```bash
# Test API key directly
curl -H "Authorization: Bearer YOUR_KEY" \
  https://api.atlascloud.ai/v1/models

# Check base URL configuration
openclaw config get env.ATLAS_API_BASE
# Should return: "https://api.atlascloud.ai/v1/"

# Check model configuration
openclaw config get env.ATLAS_API_MODEL
# Should return: "zai-org/glm-4.7"
```

### Discord Integration Issues

**Issue:** Bot not responding to messages

**Checklist:**
- [ ] MESSAGE CONTENT INTENT enabled
- [ ] Bot invited to server
- [ ] Bot token correct
- [ ] Gateway running

**Debug:**
```bash
# Check Discord channel status
openclaw channels status discord

# Check gateway logs
railway logs
```

**Issue:** "disallowed intents" error

**Solution:**
1. Go to Discord Developer Portal
2. Your App → Bot → Privileged Gateway Intents
3. Enable MESSAGE CONTENT INTENT
4. Save Changes
5. Restart Openclaw gateway

### Telegram Integration Issues

**Issue:** Bot can't read messages

**Solution:**
```
In BotFather:
/newbot
/setprivacy → Disabled
```

**Issue:** Bot commands not working

**Solution:**
```bash
# Check bot permissions
# In BotFather:
/mybots
→ Select your bot
→ Bot Settings → Allow Groups? (if needed)
```

### Getting Help

If issues persist:

1. **Check Logs**
   ```bash
   railway logs
   # Or in container:
   tail -f /var/log/openclaw.log
   ```

2. **Run Diagnostics**
   ```bash
   openclaw doctor
   ```

3. **Community Resources**
   - [Openclaw GitHub Issues](https://github.com/openclaw/openclaw/issues)
   - [Openclaw Documentation](https://docs.openclaw.ai/)
   - [Discord Community](https://discord.gg/openclaw)

4. **Reset Configuration**
   ```
   Visit: /setup
   Click: "Reset Configuration"
   Re-run setup wizard
   ```

---

## Progress Tracker

Use this checklist to track your progress through the tutorial:

### Phase 1: Deployment
- [ ] Prerequisites completed
- [ ] Railway account created
- [ ] Atlas Cloud API key obtained
- [ ] Repository cloned/forked
- [ ] Railway project deployed
- [ ] Volume configured at `/data`
- [ ] `SETUP_PASSWORD` set
- [ ] Health check passing

### Phase 2: Configuration
- [ ] Setup wizard completed
- [ ] AI provider configured
- [ ] Gateway running
- [ ] Control UI accessible

### Phase 3: Integrations
- [ ] Atlas Cloud working
- [ ] Discord bot configured (optional)
- [ ] Telegram bot configured (optional)
- [ ] Test messages sent and received

### Phase 4: Validation
- [ ] Chat test successful
- [ ] API key verified
- [ ] Configuration validated
- [ ] Integration tested

### Phase 5: Advanced (Optional)
- [ ] Skills installed
- [ ] Web search configured
- [ ] Custom agents created
- [ ] Security hardening applied

---

## Knowledge Check

Test your understanding:

<details>
<summary>1. Why does Openclaw use "venice" as the provider name for Atlas Cloud?</summary>

**Answer:** Venice AI and Atlas Cloud share infrastructure. Openclaw's internal provider ID is "venice" for this service. The `--venice-api-key` flag is used during onboarding.
</details>

<details>
<summary>2. What file stores agent authentication profiles?</summary>

**Answer:** `/data/.openclaw/agents/main/agent/auth-profiles.json`

The format is:
```json
{
  "default": {
    "provider": "venice",
    "apiKey": "sk-..."
  }
}
```
</details>

<details>
<summary>3. What privileged gateway intent is required for Discord?</summary>

**Answer:** MESSAGE CONTENT INTENT

This is critical - without it, the bot cannot read message content.
</details>

<details>
<summary>4. How do you reset the Openclaw configuration?</summary>

**Answer:** Visit `/setup` and click "Reset Configuration" to delete the config file and re-run the wizard.
</details>

<details>
<summary>5. What environment variable must be set for Railway deployment?</summary>

**Answer:** `SETUP_PASSWORD` - This protects the `/setup` wizard and is also used as the gateway authentication token.
</details>

---

## Next Steps

After completing this tutorial:

1. **Explore Openclaw Skills**
   - Browse available skills: https://github.com/openclaw/openclaw/discussions
   - Install a skill: `openclaw skills install @openclaw/skill-[name]`

2. **Configure Web Search**
   - Get Brave API key: https://brave.com/search/api/
   - Set in config: `openclaw config set tools.web.braveApiKey YOUR_KEY`

3. **Join the Community**
   - Discord: [Openclaw Discord](https://discord.gg/openclaw)
   - GitHub: [Openclaw Discussions](https://github.com/openclaw/openclaw/discussions)

4. **Advanced Topics**
   - Multi-agent setups for specialized tasks
   - Custom skill development
   - Performance optimization

5. **Stay Updated**
   - Watch for new releases: https://github.com/openclaw/openclaw/releases
   - Read CHANGELOG: https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md

---

## References

### Official Documentation
- **Openclaw GitHub**: https://github.com/openclaw/openclaw
- **Openclaw Releases**: https://github.com/openclaw/openclaw/releases
- **CHANGELOG**: https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md
- **Documentation**: https://docs.openclaw.ai/
- **Providers Guide**: https://docs.openclaw.ai/providers

### Atlas Cloud Resources
- **Atlas Cloud**: https://www.atlascloud.ai/
- **Documentation**: https://www.atlascloud.ai/docs
- **API Reference**: https://www.atlascloud.ai/docs/en/models/get-start
- **Model List**: https://www.atlascloud.ai/docs/en/models/llm

### Platform Documentation
- **Railway**: https://railway.app/docs
- **Discord Developers**: https://discord.com/developers/applications
- **Telegram BotFather**: https://t.me/BotFather

### Community
- **Openclaw Discord**: https://discord.gg/openclaw
- **Openclaw Reddit**: https://reddit.com/r/OpenClaw
- **Twitter/X**: [@OpenClawAI](https://twitter.com/OpenClawAI)

---

## Tutorial Feedback

Found an issue with this tutorial? Have suggestions for improvement?

1. **Report Issues**: https://github.com/[your-username]/openclaw-railway-template-easy-config/issues
2. **Submit PR**: Improve the tutorial directly
3. **Discussions**: Start a discussion on GitHub

---

**Version:** 1.0
**Last Updated:** 2026-02-01
**Openclaw Version:** 2026.1.30+
