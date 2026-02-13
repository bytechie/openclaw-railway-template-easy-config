# Magic Patterns MCP Integration with OpenClaw

> A comprehensive guide to integrating Magic Patterns AI design generation with OpenClaw using the Model Context Protocol (MCP).

## Table of Contents

1. [Overview](#overview)
2. [Why MCP Over API?](#why-mcp-over-api)
3. [Prerequisites](#prerequisites)
4. [Installation](#installation)
5. [Configuration](#configuration)
6. [Usage Examples](#usage-examples)
7. [Railway Deployment](#railway-deployment)
8. [Troubleshooting](#troubleshooting)

---

## Overview

### What is Magic Patterns MCP?

**Magic Patterns MCP** is a Model Context Protocol server that allows AI assistants to generate UI designs programmatically. Unlike the REST API, MCP requires **no API key** and is designed specifically for AI agent integrations.

### Why MCP for OpenClaw?

OpenClaw has **native MCP support**, making this the cleanest integration path:

| Benefit | Description |
|---------|-------------|
| **No API Key Required** | Uses your Magic Patterns account directly |
| **Native Integration** | OpenClaw supports MCP out of the box |
| **AI-Optimized** | Tools designed for AI agent usage |
| **Simple Setup** | No custom skill code needed |
| **Auto-Discovery** | OpenClaw discovers available tools automatically |

### Supported MCP Tools

The Magic Patterns MCP server provides these tools:

| Tool | Description |
|------|-------------|
| `create_design` | Generate a UI design from text description |
| `get_design_status` | Check the status of a design generation |
| `get_design_result` | Retrieve the generated design (preview URL, source files) |

---

## Important: Claude Code vs OpenClaw Authentication

**This is a critical distinction** that often causes confusion:

| Aspect | Claude Code Desktop | OpenClaw Gateway |
|--------|-------------------|------------------|
| **MCP Command** | `/mcp` (built-in) | No `/mcp` command needed |
| **Auth Method** | OAuth via `/mcp` command | URL-based config only |
| **Setup Location** | Claude Code Settings | `openclaw.json` config file |
| **When to use** | Local development with Claude Code | Server/Agent deployments (Railway, Docker) |
| **Configuration** | Run `/mcp` in Claude Code to trigger OAuth | Set URL in OpenClaw config |

### The `/mcp` Command (Claude Code Only)

If you see documentation mentioning **"Run `/mcp` inside Claude Code to trigger OAuth authentication"**, this applies to **Claude Code desktop app only**, NOT OpenClaw.

```bash
# This ONLY works in Claude Code desktop (claude.ai/code)
/mcp  # Triggers OAuth flow for Claude Desktop

# This does NOT exist in OpenClaw
# OpenClaw uses URL-based configuration instead
```

### OpenClaw MCP Authentication (No OAuth Required)

For OpenClaw deployments (including Railway), authentication works differently:

**Anonymous Access** (default - works for all OpenClaw deployments):
```bash
# This is all you need for OpenClaw
openclaw config set mcp.servers.magicpatterns.url="https://mcp.magicpatterns.com/mcp"
openclaw gateway restart
```

- No OAuth required
- No `/mcp` command needed
- Works immediately
- Sufficient for UI design generation

**When to use which:**
- **OpenClaw Gateway / Railway**: Use URL-based configuration (this guide)
- **Claude Code Desktop**: Use `/mcp` command for OAuth
- **Both can coexist**: If you use both tools, configure them separately

---

## Why MCP Over API?

| Aspect | MCP (Recommended) | API |
|--------|-------------------|-----|
| **Authentication** | Account-based (no API key) | API key required |
| **Setup Complexity** | Simple config change | Custom skill development |
| **Code Required** | None | TypeScript skill implementation |
| **Maintenance** | Updates via MCP server | Manual skill updates |
| **Cost** | Included with account | Separate API pricing |
| **OpenClaw Integration** | Native support | Custom wrapper needed |

**Bottom Line:** Use MCP for OpenClaw integration unless you have a specific reason to use the REST API.

---

## Prerequisites

### Required Accounts

1. **Magic Patterns Account**
   - Sign up at: https://www.magicpatterns.com
   - Free tier includes MCP access
   - No payment method required for MCP
   - **Note**: For OpenClaw, you don't need to authenticate via OAuth - just configure the URL

2. **OpenClaw Installation**
   - OpenClaw must be installed and running
   - Gateway must be accessible
   - **No `/mcp` command**: OpenClaw does not use Claude Code's `/mcp` command

### Verify MCP Support

Check if your OpenClaw version supports MCP:

```bash
openclaw --version
# MCP support added in OpenClaw 1.5.0+
```

---

## Installation

There are two ways to connect to Magic Patterns MCP:

### Option A: Hosted MCP Server (Recommended)

The official Magic Patterns hosted MCP server requires no installation:

```bash
# Add hosted MCP server to OpenClaw config
openclaw config set mcp.servers.magicpatterns.url="https://mcp.magicpatterns.com/mcp"
```

**Advantages:**
- No npm package installation required
- Managed by Magic Patterns team
- Always up-to-date
- Faster setup

### Option B: Local MCP Server (via npm)

Run the MCP server locally using npx:

```bash
# Test the MCP server
npx -y magic-patterns-mcp

# Add local MCP server to OpenClaw config
openclaw config set mcp.servers.magicpatterns.command="npx"
openclaw config set mcp.servers.magicpatterns.args="-y" "magic-patterns-mcp"
```

> **Note:** The package name is `magic-patterns-mcp` from npm.
>
> **Source:** [GitHub - ryanleecode/magic-patterns-mcp](https://github.com/ryanleecode/magic-patterns-mcp)

**Advantages:**
- Full control over server execution
- Works in restricted network environments
- No external HTTP calls after initial npx fetch

### Step 2: Verify Configuration

Check that the MCP server is configured:

```bash
openclaw config get mcp.servers
```

Expected output (Option A - Hosted):
```json
{
  "magicpatterns": {
    "url": "https://mcp.magicpatterns.com/mcp"
  }
}
```

Expected output (Option B - Local):
```json
{
  "magicpatterns": {
    "command": "npx",
    "args": ["-y", "magic-patterns-mcp"]
  }
}
```

### Step 4: Restart OpenClaw Gateway

For the MCP server to be loaded, restart the gateway:

```bash
# Restart the gateway
openclaw gateway restart

# Or if running via Railway
# The gateway will restart automatically
```

---

## Configuration

### Environment Variables

No API key is required for MCP. The server uses your Magic Patterns account session.

### OpenClaw MCP Configuration

The MCP server is configured via OpenClaw's config file:

**Location:** `~/.openclaw/openclaw.json` (or `/data/.openclaw/openclaw.json` on Railway)

**Config structure (Option A - Hosted, Recommended):**
```json
{
  "mcp": {
    "servers": {
      "magicpatterns": {
        "url": "https://mcp.magicpatterns.com/mcp"
      }
    }
  }
}
```

**Config structure (Option B - Local):**
```json
{
  "mcp": {
    "servers": {
      "magicpatterns": {
        "command": "npx",
        "args": ["-y", "magic-patterns-mcp"]
      }
    }
  }
}
```

### Railway-Specific Configuration

For Railway deployment, the wrapper can automatically configure MCP.

**Option A: Hosted MCP Server (Recommended)**

```javascript
// After onboarding, configure MCP (hosted)
await exec('openclaw config set mcp.servers.magicpatterns.url="https://mcp.magicpatterns.com/mcp"');
```

**Option B: Local MCP Server (via npx)**

```javascript
// After onboarding, configure MCP (local)
await exec('openclaw config set mcp.servers.magicpatterns.command="npx"');
await exec('openclaw config set mcp.servers.magicpatterns.args="-y" "magic-patterns-mcp"');
```

---

## Usage Examples

### Example 1: Generate a Login Page

**User Request:**
```
Create a login page with email and password fields, remember me checkbox,
and social login buttons for Google and GitHub
```

**OpenClaw Agent (with MCP) automatically:**
1. Calls `create_design` tool with the prompt
2. Waits for generation
3. Returns preview URL and source code

**Response:**
```
I've generated a login page design for you!

Preview: https://abc123-preview.magicpatterns.app
Editor: https://www.magicpatterns.com/c/abc123

The design includes:
- Centered card layout
- Email input with validation
- Password field with visibility toggle
- Remember me checkbox
- Social login buttons (Google, GitHub)
- Minimalist design with blue accents
```

### Example 2: Generate a Dashboard

**User Request:**
```
Build an admin dashboard with sidebar navigation, stats cards,
and a data table showing recent transactions
```

**Agent Workflow:**
```
1. Understanding requirements... OK
2. Calling create_design tool...
   Prompt: "Admin dashboard with sidebar navigation (Dashboard, Users,
   Settings, Reports), 4 statistics cards at the top (Total Users,
   Revenue, Active Sessions, Conversion Rate), and a data table showing
   recent transactions with columns for ID, User, Amount, Date, and Status"
3. Design generated successfully!
4. Retrieving result...

Your dashboard is ready:
Preview: https://xyz789-preview.magicpatterns.app
```

### Example 3: Iterative Design

**User Request:**
```
Can you update the design to use a dark theme?
```

**Agent Workflow:**
```
1. Generating new design variant with dark theme...
2. Preview: https://dark123-preview.magicpatterns.app

The design now features:
- Dark background (#0f172a)
- Light text for contrast
- Accent colors adjusted for dark mode
- All components updated for dark theme
```

---

## Railway Deployment

### Automatic MCP Configuration

The Railway wrapper can configure MCP during setup.

**Option A: Hosted MCP Server (Recommended)**

```javascript
// In the onboarding handler, after successful setup
async function configureMagicPatternsMCP(stateDir) {
  logger.info('[magicpatterns] Configuring MCP server...');

  try {
    await exec('openclaw config set mcp.servers.magicpatterns.url="https://mcp.magicpatterns.com/mcp"');
    logger.info('[magicpatterns] MCP server configured successfully (hosted)');
  } catch (error) {
    logger.error('[magicpatterns] Failed to configure MCP:', error);
  }
}
```

**Option B: Local MCP Server (via npx)**

```javascript
// In the onboarding handler, after successful setup
async function configureMagicPatternsMCP(stateDir) {
  logger.info('[magicpatterns] Configuring MCP server...');

  try {
    await exec('openclaw config set mcp.servers.magicpatterns.command="npx"');
    await exec('openclaw config set mcp.servers.magicpatterns.args="-y" "magic-patterns-mcp"');
    logger.info('[magicpatterns] MCP server configured successfully (local)');
  } catch (error) {
    logger.error('[magicpatterns] Failed to configure MCP:', error);
  }
}
```

**2. Call during onboarding:**

```javascript
// After successful onboarding
await configureMagicPatternsMCP(process.env.OPENCLAW_STATE_DIR);
```

### Verify MCP is Loaded

After deployment, check that MCP tools are available:

```bash
# SSH into Railway container
railway shell

# Check MCP servers
openclaw config get mcp.servers

# Check available tools (if OpenClaw supports this)
openclaw mcp list-tools
```

### No Environment Variables Needed

Unlike the API approach, **no `MAGICPATTERNS_API_KEY` is required** for MCP.

---

## Troubleshooting

### Issue: MCP tools not available

**Symptoms:** Agent doesn't use Magic Patterns tools

**Solutions:**
```bash
# Check MCP server configuration
openclaw config get mcp.servers

# Verify configuration exists
openclaw config get mcp.servers.magicpatterns

# Restart gateway to reload MCP servers
openclaw gateway restart
```

### Issue: "npx: command not found"

**Cause:** npx not available in the environment

**Solution:** Ensure Node.js and npm are installed:

```bash
# Check Node.js version
node --version

# Check npm version
npm --version

# If missing, install Node.js
```

### Issue: MCP server fails to start

**Symptoms:** Gateway logs show MCP connection errors

**Solutions (for Local/Option B):**
```bash
# Test MCP server manually
npx magic-patterns-mcp

# Check network connectivity
curl -I https://www.magicpatterns.com

# Verify npm package exists
npm view magic-patterns-mcp
```

**Solutions (for Hosted/Option A):**
```bash
# Test connectivity to hosted MCP
curl -I https://mcp.magicpatterns.com/mcp

# Verify URL is correct in config
openclaw config get mcp.servers.magicpatterns.url
```

### Issue: Design generation fails

**Possible Causes:**
- Invalid prompt format
- Service temporarily unavailable
- Account limits reached

**Solutions:**
1. **Retry with a simpler prompt**
2. **Check Magic Patterns account status**
3. **Verify the prompt describes a UI design**

### Issue: "module not found" error

**Cause:** OpenClaw can't find the MCP server package (Local/Option B only)

**Solution:** Ensure the package is installed:

```bash
# For local development
npm install -g magic-patterns-mcp

# For Railway (Dockerfile)
RUN npm install -g magic-patterns-mcp
```

**Alternative:** Switch to hosted URL-based MCP (Option A) which requires no local installation:

```bash
openclaw config set mcp.servers.magicpatterns.url="https://mcp.magicpatterns.com/mcp"
```

### Debug Mode

Enable verbose logging to troubleshoot MCP issues:

```bash
# Enable debug logging
openclaw config set logging.level="debug"

# Restart gateway
openclaw gateway restart

# Check logs for MCP-related messages
openclaw gateway logs
```

---

## Best Practices

### Prompt Design for Better Results

| Practice | Example |
|----------|---------|
| **Be specific** | "Create a login page with email and password fields" vs "Make a login" |
| **Mention layout** | "with sidebar navigation" vs "with navigation" |
| **Specify components** | "email input, password field with visibility toggle" |
| **Describe style** | "minimalist design with blue accent colors" |
| **Include functionality** | "with client-side validation" |

### Iterative Workflow

```
1. Generate initial design (quick iteration)
2. Review preview URL
3. Request changes (refine as needed)
4. Generate final version
5. Export source code from Magic Patterns editor
```

### Cost Optimization

- **MCP is included** with Magic Patterns account
- **No separate API costs** like the REST API
- **No rate limit concerns** for typical usage

---

## Comparison: MCP vs API

### When to Use MCP

- OpenClaw agent integration
- AI-assisted UI generation
- No separate API infrastructure needed
- Simpler setup and maintenance

### When to Use API

- Non-AI applications
- Need for fine-grained control
- Custom authentication flows
- Batch processing without AI involvement

**For OpenClaw integration, MCP is the recommended approach.**

---

## Advanced Configuration

### Custom Presets

MCP supports all Magic Patterns presets:

| Preset | Best For |
|--------|----------|
| `html-tailwind` | General web apps |
| `shadcn-tailwind` | Modern React apps |
| `chakraUi-inline` | React with Chakra UI |
| `mantine-inline` | React with Mantine |
| Custom | Your design system |

Specify preset in prompt:
```
Create a dashboard using shadcn-tailwind preset with...
```

### Custom Design Systems

1. **Create custom preset** in Magic Patterns
2. **Reference by ID** in prompts:
   ```
   Create a product page using preset "my-brand-design"
   ```

---

## References

### Magic Patterns Resources

- **Website:** https://www.magicpatterns.com
- **MCP Documentation:** https://www.magicpatterns.com/docs/documentation/features/mcp-server/overview
- **MCP Server Package (Local):** https://www.npmjs.com/package/magic-patterns-mcp
- **GitHub (Local MCP):** https://github.com/ryanleecode/magic-patterns-mcp
- **Account:** https://www.magicpatterns.com/settings

### OpenClaw Resources

- **MCP Documentation:** https://docs.openclaw.ai/tools/mcp
- **Configuration Guide:** https://docs.openclaw.ai/configuration
- **ClawHub (Skills):** https://clawhub.com

### Model Context Protocol

- **MCP Specification:** https://modelcontextprotocol.io
- **MCP SDK:** https://github.com/modelcontextprotocol

---

## FAQ

### Do I need a Magic Patterns account for MCP?

Yes, but **no API key is required**. Sign up at https://www.magicpatterns.com for free.

### Do I need to run `/mcp` command for OpenClaw?

**No!** The `/mcp` command is **specific to Claude Code desktop app only**.

OpenClaw uses URL-based configuration:

```bash
openclaw config set mcp.servers.magicpatterns.url="https://mcp.magicpatterns.com/mcp"
```

This is all you need - no OAuth, no `/mcp` command required.

### What's the difference between Claude Code and OpenClaw?

- **Claude Code Desktop**: AI coding assistant from Anthropic, uses `/mcp` command for OAuth
- **OpenClaw**: Gateway platform for AI agents, uses URL-based MCP configuration

Different tools, different setup methods!

### Is MCP free to use?

MCP access is **included with your Magic Patterns account**. There's no separate API pricing.

### Can I use both MCP and API?

Yes, but for OpenClaw integration, MCP is recommended for simplicity.

### What if MCP doesn't meet my needs?

If you need features not available via MCP, the REST API provides full control at the cost of additional setup complexity.

---

**Version:** 1.1
**Last Updated:** 2026-02-12
**Magic Patterns MCP Version:** Latest (Hosted: <https://mcp.magicpatterns.com/mcp> or via npm `magic-patterns-mcp`)
