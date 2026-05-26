---
name: uninstall
description: "Remove all Jira Planner config and restore pre-install state."
---

# Uninstall Guide

## For Agents — Interactive Uninstall Protocol

**You are guiding a human through uninstallation. Follow this script step by step.**

- ONE step at a time. Wait for user response before next step.
- YOU perform all actions. The user only confirms.
- If user says "Uninstall Jira Planner", start from Step 1.

---

### Step 0: Confirm Intent

**Say to user:**

```text
Jira Planner uninstall starting.

The following will be removed:
1. MCP server (atlassian) — from ~/.codex/config.toml
2. Guardrail Hooks — from hooks.json

Workspace files ($JIRA_PLANNER_SPACE_DIR) and plan data are preserved.
You can delete them manually if needed.

Proceed? (Preservation options are also available)
```

**Wait for user confirmation.**

**If user asks about options, present:**

```text
Preservation options:
- Keep MCP: If you use the Atlassian MCP for other purposes
```

---

### Step 1: Dry Run

Run the uninstaller in dry-run mode first to show what will be removed:

```bash
bash scripts/uninstall.sh --dry-run
```

**Show output to user and say:**

```text
The above items will be removed. Proceed?
```

**Wait for confirmation.**

---

### Step 2: Execute Uninstall

Run the uninstaller with user-selected flags:

```bash
# No options (full removal)
bash scripts/uninstall.sh

# With options (example)
bash scripts/uninstall.sh --keep-mcp
```

---

### Step 3: API Token Warning

**If MCP was removed (no --keep-mcp), say to user:**

```text
Atlassian API Token has been removed from ~/.codex/config.toml.

For security, also revoke the token in Atlassian:
https://id.atlassian.com/manage-profile/security/api-tokens

(You can generate a new token if you reinstall Jira Planner later.)
```

---

### Step 4: Verify & Restart

**Say to user:**

```text
Uninstall complete!

Please restart Codex CLI.

To fully remove workspace files:
  rm -rf $JIRA_PLANNER_SPACE_DIR

To reinstall, say "Install Jira Planner" to start again.
```

---

## What Gets Removed

| File | Removed | Notes |
|------|---------|-------|
| `~/.codex/config.toml` → `mcpServers.atlassian` | Yes (unless --keep-mcp) | API token included |
| Plugin hooks configuration | Yes | Other hooks preserved |

## What Stays

| Path | Reason |
|------|--------|
| `$JIRA_PLANNER_SPACE_DIR` | Plan files — user deletes manually |
| Plugin directory | Managed by Codex plugin system |
