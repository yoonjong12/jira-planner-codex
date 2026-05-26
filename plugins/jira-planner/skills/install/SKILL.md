---
name: install
description: "Set up Jira MCP and connect to workspace. Interactive installation guide."
---

# Installation Guide

## For Agents — Interactive Installation Protocol

**You are guiding a human through installation. Follow this script step by step.**

- ONE step at a time. Wait for user response before next step.
- YOU perform all file edits. The user only provides information.
- If user says "Install Jira Planner", start from Step 1.

---

### Step 1: API Token

**Say to user:**

```text
Jira Planner installation starting.

An Atlassian API Token is required.
Generate one at the link below and paste it here:

https://id.atlassian.com/manage-profile/security/api-tokens

(Token does not expire until manually revoked.)
```

**Wait for user to paste token.**

**Then ask:**

```text
Please also provide your Atlassian account email. (e.g., name@company.com)
```

**Collect:** `API_TOKEN` and `EMAIL` from user.

---

### Step 2: Configure Atlassian MCP (Agent does this)

**DO NOT ask user to edit files. You do it.**

Read `~/.codex/config.toml`, then add the Atlassian MCP server configuration.

For Codex CLI, MCP servers are configured in `~/.codex/config.toml` under `[mcpServers]`:

```toml
[mcpServers.atlassian]
command = "uvx"
args = ["mcp-atlassian"]

[mcpServers.atlassian.env]
JIRA_URL = "https://mindai.atlassian.net"
JIRA_USERNAME = "<email>"
JIRA_API_TOKEN = "<token>"
CONFLUENCE_URL = "https://mindai.atlassian.net/wiki"
CONFLUENCE_USERNAME = "<email>"
CONFLUENCE_API_TOKEN = "<token>"
```

Also install `markdown-reader` scripts to the plugin's scripts directory (already included in this plugin at `scripts/markdown-reader/`).

**Say to user:**

```text
Configuration complete. Please restart Codex CLI.

After restart, say "continue install" to proceed to the verification step.
```

---

### Step 3: Verify Connection (after restart)

**Ask user for default project key:**

```text
Please enter the Jira project key to use (e.g., WAO, PROJ, DEV):
```

**Collect:** `PROJECT_KEY` from user.

**Agent performs these calls silently:**

```typescript
mcp__atlassian__jira_search({
  jql: `project = ${PROJECT_KEY} AND type = Epic ORDER BY updated DESC`,
  limit: 1,
  fields: "summary,status"
})
```

**If succeeds, say:**

```text
Connection verified!

  Workspace: mindai.atlassian.net
  Project: ${PROJECT_KEY}
  Auth: API Token (permanent)

Next: Installing guardrail hooks.
```

**If fails, troubleshoot:**
- 401 → API token or email error. Request re-entry.
- 404 → Verify project key or check access permissions.
- Connection error → Check `uvx mcp-atlassian` package installation.

---

### Step 4: Install Guardrail Hooks (Agent does this)

Run the installer:

```bash
bash scripts/install-hooks.sh
```

This sets up:
- Jira description template validator
- Plan file format validator

**Say to user:**

```text
Guardrail hooks installed.

Installed safeguards:
- Jira description template validation (Context/Objective/Deliverables/AC)
- Plan file format validation

Jira Planner installation complete!
After restart, use jira-planner:help to get started.
```

---

## Verification Checklist

Agent confirms all of these during Step 3-4:

- [x] `~/.codex/config.toml` has atlassian in mcpServers (with API token)
- [x] Can fetch issues (`mcp__atlassian__jira_get_issue`)
- [x] Can search issues (`mcp__atlassian__jira_search`)
- [x] Guardrail hooks configured

## Troubleshooting

### Authentication Failed (401)

1. Verify API token is correct (regenerate if needed)
2. Check email matches your Atlassian account
3. Ensure `JIRA_URL` is `https://mindai.atlassian.net`

### MCP Server Not Appearing

1. Verify `~/.codex/config.toml` has the atlassian entry in `[mcpServers]`
2. Restart Codex CLI completely
3. Check for TOML syntax errors

### No Projects Visible

1. Verify you have project access in Atlassian
2. Check with workspace admin for permissions

## References

- [mcp-atlassian (sooperset)](https://github.com/sooperset/mcp-atlassian)
