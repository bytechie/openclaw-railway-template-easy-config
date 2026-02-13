# Magic Patterns MCP Testing Guide

> Simple test cases to verify Magic Patterns MCP integration with OpenClaw on Railway.

## Table of Contents

1. [Quick Test Overview](#quick-test-overview)
2. [Prerequisites](#prerequisites)
3. [Test Case 1: Configuration Verification](#test-case-1-configuration-verification)
4. [Test Case 2: Simple Design Generation](#test-case-2-simple-design-generation)
5. [Test Case 3: Design with Presets](#test-case-3-design-with-presets)
6. [Test Case 4: Iterative Design Update](#test-case-4-iterative-design-update)
7. [Test Case 5: Multi-Model Verification](#test-case-5-multi-model-verification)
8. [Troubleshooting Failed Tests](#troubleshooting-failed-tests)

---

## Quick Test Overview

### What We're Testing

| Component | How We Test It |
|-----------|-----------------|
| **MCP Server Connection** | Verify config exists and gateway loads it |
| **Tool Availability** | Check OpenClaw recognizes Magic Patterns tools |
| **Design Generation** | Create a simple UI component |
| **Model Integration** | Verify AI model can call MCP tools |
| **End-to-End Flow** | Full request → design → preview flow |

### Test Levels

- **Level 1**: Configuration only (30 seconds) - Verify MCP is configured
- **Level 2**: Basic generation (2 minutes) - Create a simple design
- **Level 3**: Full integration (5 minutes) - Test with AI model and multiple tools

---

## Prerequisites

### Required

1. **Railway deployment running** with green status
2. **Setup wizard completed** at least once
3. **Atlas Cloud API key** configured (or another model provider)

### Check Before Testing

```bash
# 1. Verify Railway deployment is running
# Go to your Railway project and check service status is "Running"

# 2. Check you can access the gateway
curl -I https://your-app.up.railway.app/setup/healthz

# 3. Verify setup password works
curl -u :your-password https://your-app.up.railway.app/setup/api/status
```

### Expected Environment

```bash
# These should be set in Railway Variables
SETUP_PASSWORD=your-secure-password
OPENCLAW_STATE_DIR=/data/.openclaw
OPENCLAW_WORKSPACE_DIR=/data/workspace
OPENAI_API_KEY=your-atlas-cloud-key  # or other provider
```

---

## Test Case 1: Configuration Verification

**Time**: 30 seconds | **Level**: Basic

### Objective

Verify Magic Patterns MCP server is configured in OpenClaw.

### Steps

1. **SSH into Railway container**:
   ```bash
   railway shell
   ```

2. **Check MCP configuration**:
   ```bash
   openclaw config get mcp.servers
   ```

3. **Verify Magic Patterns entry**:
   ```bash
   openclaw config get mcp.servers.magicpatterns
   ```

### Expected Results

```json
{
  "magicpatterns": {
    "url": "https://mcp.magicpatterns.com/mcp"
  }
}
```

### Pass Criteria

- ✅ `mcp.servers.magicpatterns` exists
- ✅ URL is `https://mcp.magicpatterns.com/mcp`
- ✅ No error messages

### Fail Scenarios

| Error | Cause | Solution |
|--------|--------|----------|
| `Property not found` | MCP not configured | Run setup wizard again |
| `Invalid URL` | Wrong URL format | Check configuration in `src/server.js` |
| `Command not found` | OpenClaw not installed | Rebuild Docker container |

---

## Test Case 2: Simple Design Generation

**Time**: 2 minutes | **Level**: Basic

### Objective

Generate a simple UI component using Magic Patterns MCP.

### Steps

#### Option A: Test via Railway Shell

1. **SSH into Railway container**:
   ```bash
   railway shell
   ```

2. **List available MCP tools** (if OpenClaw supports this):
   ```bash
   openclaw mcp list-tools

   # Look for: magicpatterns.create_design
   ```

3. **Test with OpenClaw CLI**:
   ```bash
   echo "Create a simple login page with email and password fields" | openclaw chat
   ```

#### Option B: Test via Web UI

1. **Access OpenClaw UI**:
   ```
   https://your-app.up.railway.app/openclaw
   ```

2. **Send test prompt**:
   ```
   Create a simple button component with hover effect
   ```

3. **Wait for response** (30-60 seconds)

### Expected Results

The agent should:

1. **Acknowledge the request**
2. **Call Magic Patterns MCP tool** (visible in logs)
3. **Return design URLs**:
   ```
   Preview: https://abc123-preview.magicpatterns.app
   Editor: https://www.magicpatterns.com/c/abc123
   ```

4. **Optionally show code snippets**

### Pass Criteria

- ✅ Agent calls `create_design` or similar MCP tool
- ✅ Preview URL is returned and accessible
- ✅ Design matches prompt description
- ✅ No error messages in response

### Verify Design

```bash
# Test the preview URL works
curl -I https://abc123-preview.magicpatterns.app

# Expected: HTTP/1.1 200 OK
```

---

## Test Case 3: Design with Presets

**Time**: 3 minutes | **Level**: Intermediate

### Objective

Generate a design using specific preset (e.g., shadcn-tailwind).

### Steps

1. **Access OpenClaw UI** or use CLI

2. **Send prompt with preset specification**:
   ```
   Create a modern dashboard card component using shadcn-tailwind preset.
   The card should show:
   - User avatar on the left
   - Name and title
   - "View Profile" button
   - Minimalist design with gray background
   ```

3. **Wait for generation**

### Expected Results

```
Generated dashboard card component:
Preview: https://xyz789-preview.magicpatterns.app
Editor: https://www.magicpatterns.com/c/xyz789

Features:
- shadcn/ui Card component
- Avatar, name, title layout
- Button with hover state
- Tailwind CSS styling
```

### Pass Criteria

- ✅ Design uses specified preset style
- ✅ All requested components present
- ✅ Preview URL is accessible
- ✅ Code follows Tailwind/shadcn conventions

---

## Test Case 4: Iterative Design Update

**Time**: 5 minutes | **Level**: Advanced

### Objective

Test the `update_design` tool by modifying an existing design.

### Steps

1. **Generate initial design**:
   ```
   Create a login form with email and password fields
   ```

2. **Note the design URL** from response

3. **Request update**:
   ```
   Update the login form to add:
   - "Remember me" checkbox
   - "Forgot password" link
   - Change button color to blue (#3b82f6)
   ```

### Expected Results

```
Design updated successfully:
Preview: https://abc123-preview.magicpatterns.app (same URL)
Request ID: req_456

Changes applied:
- Added checkbox component
- Added link component
- Updated button styling
```

### Pass Criteria

- ✅ Agent calls `update_design` tool
- ✅ Same preview URL (design updated in-place)
- ✅ Requested changes visible in preview
- ✅ No duplicate designs created

---

## Test Case 5: Multi-Model Verification

**Time**: 5 minutes | **Level**: Advanced

### Objective

Verify different AI models can use Magic Patterns MCP tools.

### Prerequisites

Atlas Cloud with multiple models configured.

### Steps

#### Test 5.1: GLM-4.7

1. **Switch model** (if needed):
   ```bash
   railway shell
   openclaw config set agents.defaults.model.primary="atlas/zai-org/glm-4.7"
   openclaw gateway restart
   ```

2. **Generate design**:
   ```
   Create a Chinese-language login page: 请创建一个登录页面，包含邮箱和密码输入框
   ```

#### Test 5.2: DeepSeek R1

1. **Switch model**:
   ```bash
   openclaw config set agents.defaults.model.primary="atlas/deepseek-ai/deepseek-r1"
   openclaw gateway restart
   ```

2. **Generate design**:
   ```
   Create a navigation bar with logo, menu items, and user dropdown
   ```

### Expected Results

| Model | Expected Behavior |
|--------|-----------------|
| **GLM-4.7** | Better Chinese prompts, faster generation |
| **DeepSeek R1** | More detailed analysis, structured approach |

Both models should:
- ✅ Successfully call Magic Patterns MCP tools
- ✅ Generate valid preview URLs
- ✅ Produce functional UI designs

### Pass Criteria

- ✅ Multiple models can use same MCP tools
- ✅ Each model leverages its strengths (language, reasoning)
- ✅ No model-specific errors

---

## Troubleshooting Failed Tests

### Issue: "Tool not found" or "MCP server not responding"

**Diagnose**:
```bash
railway shell
openclaw config get mcp.servers
```

**Solutions**:
1. Verify Magic Patterns entry exists
2. Check URL is correct: `https://mcp.magicpatterns.com/mcp`
3. Restart gateway:
   ```bash
   openclaw gateway restart
   ```

### Issue: Design generation times out

**Diagnose**: Check Railway logs
```bash
railway logs | grep -i magicpatterns
```

**Solutions**:
1. **Network issue**: Verify Railway can reach external URLs
   ```bash
   curl -I https://mcp.magicpatterns.com/mcp
   ```
2. **Retry with simpler prompt**: Reduce complexity
3. **Check Magic Patterns status**: Service may be temporarily down

### Issue: Agent doesn't call MCP tools

**Diagnose**:
```bash
# Check model is configured
openclaw config get agents.defaults.model.primary
```

**Solutions**:
1. **Verify model supports tool calling** (most modern models do)
2. **Be more explicit in prompt**:
   ```
   Use Magic Patterns to create a login page
   ```
3. **Check gateway logs** for tool call errors:
   ```bash
   railway logs
   ```

### Issue: Preview URL returns 404

**Diagnose**: Test the URL directly
```bash
curl -I https://abc123-preview.magicpatterns.app
```

**Solutions**:
1. **Design may still be processing**: Wait 30 seconds and retry
2. **Incorrect URL**: Verify URL format in response
3. **Design failed**: Retry generation with simpler prompt

---

## Test Checklist

Use this checklist to verify your integration:

- [ ] Test Case 1: MCP configuration verified
- [ ] Test Case 2: Simple design generated successfully
- [ ] Test Case 3: Preset design works (optional)
- [ ] Test Case 4: Design update works (optional)
- [ ] Test Case 5: Multiple models tested (optional)
- [ ] Preview URLs are accessible
- [ ] No errors in Railway logs
- [ ] Design quality meets expectations

---

## Quick Reference Test Commands

```bash
# Configuration check
openclaw config get mcp.servers.magicpatterns

# Gateway restart
openclaw gateway restart

# Test generation
echo "Create a simple button" | openclaw chat

# Check logs
railway logs | grep -i magicpatterns

# Test preview URL
curl -I https://preview-url.magicpatterns.app
```

---

## Expected Performance

| Operation | Expected Time | Notes |
|-----------|---------------|-------|
| MCP server connection | < 5 seconds | One-time on gateway start |
| Simple design (button) | 20-40 seconds | Fast mode |
| Complex design (dashboard) | 40-90 seconds | Depends on complexity |
| Design update | 30-60 seconds | Modifies existing design |

---

**Version:** 1.0
**Last Updated:** 2026-02-12
**Related Documentation:**
- [MCP Integration Guide](MAGICPATTERNS_MCP_INTEGRATION.md)
- [AI Model + MCP Integration](AI_MODEL_MCP_INTEGRATION.md)
