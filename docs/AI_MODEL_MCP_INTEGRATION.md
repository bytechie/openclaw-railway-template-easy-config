# AI Model + MCP Integration Guide

> How to use AI models (Atlas Cloud, GLM-4.7, etc.) with Magic Patterns MCP tools in OpenClaw.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Quick Start](#quick-start)
3. [Configuration Examples](#configuration-examples)
4. [Usage Examples](#usage-examples)
5. [Troubleshooting](#troubleshooting)

---

## Architecture Overview

### How Models and MCP Work Together

```
┌─────────────────────────────────────────────────────────────┐
│                   OpenClaw Gateway                        │
├─────────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────┐        ┌──────────────────────┐  │
│  │   AI Model      │        │   MCP Tools          │  │
│  │  (The "Brain")  │        │  (The "Hands")      │  │
│  └────────┬────────┘        └──────────┬───────────┘  │
│           │                            │                 │
│           │ Understands prompt          │ Executes tools  │
│           │ Decides what to do         │                 │
│           │                            │                 │
│           └────────────┬───────────────┘                 │
│                        │                                 │
│                        ▼                                 │
│              ┌──────────────────┐                       │
│              │  User Request    │                       │
│              │  "Create a     │                       │
│              │   login page"  │                       │
│              └──────────────────┘                       │
└─────────────────────────────────────────────────────────────┘
```

**Key Points:**

| Aspect | AI Model | MCP Tools |
|--------|----------|-----------|
| **Role** | "The Brain" - thinks, plans, understands | "The Hands" - executes actions |
| **Examples** | GLM-4.7, DeepSeek, GPT-4 | Magic Patterns, filesystem, database |
| **Configuration** | `models.providers.*` | `mcp.servers.*` |
| **Relationship** | **Independent** - Any model can use any MCP tool |

---

## Quick Start

### Prerequisites

1. OpenClaw installed and gateway running
2. AI model API key (Atlas Cloud, OpenAI, etc.)
3. Magic Patterns account (free, no API key needed for MCP)

### Three Simple Steps

#### Step 1: Configure AI Model

**Atlas Cloud (includes GLM-4.7):**

```bash
# Set Atlas Cloud provider
openclaw config set models.providers.atlas='{
  "baseUrl": "https://api.atlascloud.ai/v1/",
  "apiKey": "${OPENAI_API_KEY}",
  "api": "openai-completions",
  "models": [
    {"id": "zai-org/glm-4.7", "name": "Z.AI GLM-4.7"},
    {"id": "deepseek-ai/deepseek-r1", "name": "DeepSeek R1"},
    {"id": "minimaxai/minimax-m2.1", "name": "MiniMax M2.1"}
  ]
}'

# Set GLM-4.7 as primary model
openclaw config set agents.defaults.model.primary="atlas/zai-org/glm-4.7"

# Set your API key
export OPENAI_API_KEY="your-atlas-cloud-api-key"
```

#### Step 2: Configure Magic Patterns MCP

```bash
# Option A: Hosted URL (recommended)
openclaw config set mcp.servers.magicpatterns.url="https://mcp.magicpatterns.com/mcp"

# OR Option B: Local command
openclaw config set mcp.servers.magicpatterns.command="npx"
openclaw config set mcp.servers.magicpatterns.args="-y" "magic-patterns-mcp"
```

#### Step 3: Restart Gateway

```bash
openclaw gateway restart
```

---

## Configuration Examples

### Example 1: Atlas Cloud + Magic Patterns (Railway)

When using the Railway setup wizard:

1. **Select Atlas Cloud** as provider
2. **Choose GLM-4.7** as your model
3. **Complete setup** - Magic Patterns MCP is auto-configured

Your agent can now generate UI designs using GLM-4.7.

### Example 2: Multiple Models with Single MCP

```bash
# Configure multiple models
openclaw config set models.providers.atlas='{
  "baseUrl": "https://api.atlascloud.ai/v1/",
  "apiKey": "${ATLAS_API_KEY}",
  "api": "openai-completions",
  "models": [
    {"id": "zai-org/glm-4.7", "name": "GLM-4.7 - Chinese Optimized"},
    {"id": "deepseek-ai/deepseek-r1", "name": "DeepSeek R1 - Reasoning"}
  ]
}'

# Configure Magic Patterns MCP (same for all models)
openclaw config set mcp.servers.magicpatterns.url="https://mcp.magicpatterns.com/mcp"
```

Switch between models while keeping MCP tools available:

```bash
# Use GLM-4.7 for general tasks
openclaw config set agents.defaults.model.primary="atlas/zai-org/glm-4.7"

# Use DeepSeek R1 for reasoning tasks
openclaw config set agents.defaults.model.primary="atlas/deepseek-ai/deepseek-r1"
```

### Example 3: Manual Configuration File

Edit `~/.openclaw/openclaw.json` or `/data/.openclaw/openclaw.json`:

```json
{
  "models": {
    "providers": {
      "atlas": {
        "baseUrl": "https://api.atlascloud.ai/v1/",
        "apiKey": "${OPENAI_API_KEY}",
        "api": "openai-completions",
        "models": [
          {"id": "zai-org/glm-4.7", "name": "Z.AI GLM-4.7"}
        ]
      }
    },
    "mode": "merge"
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "atlas/zai-org/glm-4.7"
      }
    }
  },
  "mcp": {
    "servers": {
      "magicpatterns": {
        "url": "https://mcp.magicpatterns.com/mcp"
      }
    }
  }
}
```

---

## Usage Examples

### Example 1: GLM-4.7 Generates Login Page

**User Request:**
```
Create a login page with email and password fields
```

**Agent (using GLM-4.7) thinks:**
1. Understands the request for a login UI
2. Looks at available MCP tools
3. Finds `magicpatterns.create_design`
4. Calls the tool with appropriate prompt

**Tool Call:**
```json
{
  "tool": "create_design",
  "arguments": {
    "prompt": "Create a login page with email and password fields, minimal design with blue accents"
  }
}
```

**Response:**
```
I've generated a login page design for you!

Preview: https://abc123-preview.magicpatterns.app
Editor: https://www.magicpatterns.com/c/abc123

The design includes:
- Centered card layout
- Email input with validation
- Password field with visibility toggle
- Minimalist design with blue accents
```

### Example 2: DeepSeek R1 for Complex Dashboard

**User Request:**
```
Build an admin dashboard with user management, analytics,
and real-time data updates
```

**Agent (using DeepSeek R1) thinks:**
1. Analyzes requirements (reasoning-optimized model)
2. Plans the dashboard structure
3. Generates detailed prompt for Magic Patterns
4. Creates the design iteratively

### Example 3: Switching Models Mid-Conversation

```
User: Create a product listing page
Agent (GLM-4.7): [generates design]

User: Now add a shopping cart
Agent (GLM-4.7): [generates updated design]

User: Switch to DeepSeek for this next task - optimize the checkout flow
System: Model switched to atlas/deepseek-ai/deepseek-r1
Agent (DeepSeek R1): [analyzes and optimizes checkout flow]
```

---

## Troubleshooting

### Issue: Model works but MCP tools aren't called

**Cause:** MCP server not configured or not loaded

**Solution:**
```bash
# Check MCP configuration
openclaw config get mcp.servers

# Verify Magic Patterns is listed
openclaw config get mcp.servers.magicpatterns

# Restart gateway
openclaw gateway restart

# Check logs for MCP loading
openclaw gateway logs | grep mcp
```

### Issue: MCP tools work but model gives poor results

**Cause:** Model may lack tool-calling capability or need better prompts

**Solutions:**

1. **Check model supports tool calling** - Most modern models do
2. **Try a different model** - GLM-4.7, DeepSeek R1 are good
3. **Be more specific in prompts**

### Issue: Atlas Cloud model not available

**Solution:**
```bash
# Verify provider configuration
openclaw config get models.providers.atlas

# Check model list
openclaw config get models.providers.atlas.models

# Test API key
curl -H "Authorization: Bearer YOUR_ATLAS_KEY" \
  https://api.atlascloud.ai/v1/models
```

### Issue: "Tool not found" error

**Cause:** MCP server not connected or tool name mismatch

**Solution:**
```bash
# Check which MCP tools are available
openclaw mcp list-tools

# If magicpatterns not listed, reconfigure
openclaw config set mcp.servers.magicpatterns.url="https://mcp.magicpatterns.com/mcp"
openclaw gateway restart
```

### Important: `/mcp` Command Not Found

**Cause:** You may be looking for Claude Code's `/mcp` command

**Explanation:** The `/mcp` command is **specific to Claude Code desktop app only**, not OpenClaw.

| Tool | `/mcp` Command? | MCP Configuration |
|------|------------------|------------------|
| **Claude Code Desktop** | ✅ Yes - built-in command | OAuth via `/mcp` |
| **OpenClaw Gateway** | ❌ No such command | URL-based config only |

**For OpenClaw, use:**
```bash
# This is how OpenClaw configures MCP (no `/mcp` command needed)
openclaw config set mcp.servers.magicpatterns.url="https://mcp.magicpatterns.com/mcp"
openclaw gateway restart
```

**Cause:** MCP server not connected or tool name mismatch

**Solution:**
```bash
# Check which MCP tools are available
openclaw mcp list-tools

# If magicpatterns not listed, reconfigure
openclaw config set mcp.servers.magicpatterns.url="https://mcp.magicpatterns.com/mcp"
openclaw gateway restart
```

---

## Best Practices

### Model Selection for UI Generation

| Use Case | Recommended Model | Why |
|----------|-------------------|------|
| **Quick UI mockups** | MiniMax M2.1 | Fast, cost-effective |
| **Chinese UI text** | GLM-4.7 | Chinese-optimized |
| **Complex layouts** | DeepSeek R1 | Reasoning-optimized |
| **Large applications** | KwaiKAT Coder Pro | 256K context |
| **Production designs** | Any model with `mode: "best"` | Higher quality |

### Prompt Design for Better Results

**Good prompt for GLM-4.7:**
```
Create a login page with:
- Email input field with validation
- Password field with visibility toggle
- "Remember me" checkbox
- Social login buttons (Google, GitHub)
- Minimalist design with blue accent colors (#3b82f6)
- Centered card layout on light gray background
```

**Poor prompt:**
```
make a login
```

### Testing Your Setup

```bash
# 1. Verify model is configured
openclaw config get agents.defaults.model.primary
# Expected: "atlas/zai-org/glm-4.7"

# 2. Verify MCP is configured
openclaw config get mcp.servers.magicpatterns
# Expected: {"url": "https://mcp.magicpatterns.com/mcp"}

# 3. Test with a simple request
echo "Create a simple button component" | openclaw chat

# 4. Check gateway logs
openclaw gateway logs
```

---

## Model Compatibility

### Atlas Cloud Models

| Model | Tool Calling | Chinese | Context | Best For |
|-------|--------------|----------|----------|-----------|
| GLM-4.7 | ✅ Yes | ✅ Native | 203K | Chinese apps, general UI |
| DeepSeek R1 | ✅ Yes | ❌ No | 164K | Complex reasoning |
| MiniMax M2.1 | ✅ Yes | ❌ No | 197K | Fast iteration |
| KwaiKAT Coder Pro | ✅ Yes | ❌ No | 256K | Large codebases |

### Other Providers

All models that support **OpenAI-compatible tool calling** will work:

- ✅ OpenAI (GPT-4, GPT-4o)
- ✅ Anthropic (Claude 3.5+)
- ✅ Google (Gemini 1.5+)
- ✅ DeepSeek (V3, R1)
- ✅ Qwen (2.5 Coder)
- ✅ Any OpenAI-compatible API

---

## Railway Quick Reference

### Using Setup Wizard

1. Visit `/setup`
2. **Provider Group**: Atlas Cloud
3. **Auth Method**: Atlas Cloud API key
4. **Model**: GLM-4.7 (or your choice)
5. **Complete setup** - MCP is auto-configured

### Manual Railway Configuration

Add to Railway Variables:

```bash
OPENAI_API_KEY=your_atlas_api_key  # Atlas Cloud uses OpenAI-compatible
OPENAI_BASE_URL=https://api.atlascloud.ai/v1/
```

The Magic Patterns MCP configuration is automatically applied during onboarding.

---

## References

### OpenClaw Configuration

- **Models Documentation**: [OpenClaw Model Configuration](https://docs.openclaw.ai/configuration/models)
- **MCP Documentation**: [OpenClaw MCP Guide](https://docs.openclaw.ai/tools/mcp)

### Atlas Cloud

- **Website**: https://www.atlascloud.ai
- **API Documentation**: https://www.atlascloud.ai/docs
- **Available Models**: https://www.atlascloud.ai/models

### Magic Patterns

- **MCP Integration**: `docs/MAGICPATTERNS_MCP_INTEGRATION.md`
- **Website**: https://www.magicpatterns.com
- **MCP Server**: https://mcp.magicpatterns.com/mcp

---

**Version:** 1.0
**Last Updated:** 2026-02-12
