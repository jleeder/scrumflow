---
name: scrumflow-setup
description: |
  Initializes a ScrumFlow project. Checks for prerequisites (git, worktrees), configures model tiers, and sets up MCP ticket-sync integration. Use this to prepare a repository for ScrumFlow before implementing any features.
tools: ["read", "edit", "execute"]
---

# ScrumFlow Setup

## Purpose
Prepare the current repository for the ScrumFlow pipeline by validating the environment and generating a persistent `config.json`.

## Prerequisites Check
1. **Git Repository:** Verify the current directory is a git repository.
2. **Git Worktrees:** Check if `git worktree` is supported and functional.
3. **Branching:** Identify the base branch (usually `main` or `develop`).

## Configuration Generation

Ask the pilot for the following preferences (or suggest defaults):

### 1. Model Tiers
Which models should handle which roles?
- **Premium (PO, Architect, Reviewer):** Default to `claude-3-5-sonnet`.
- **Standard (Engineer, Red-Test):** Default to `gpt-4o` or `claude-3-haiku`.
- **Utility (Sync, Runner):** Default to a fast, cheap model.

### 2. Ticket Sync (Scrum Transparency)
- Is an MCP Issue Tracker connected?
- What is the primary Ticket Provider (Jira, Linear, GitHub)?
- What is the Project Key/Prefix?

### 3. Repository Standards
- Base branch for feature branches.
- Remote name (default: `origin`).

## Output

Write a `.scrum-flow/config.json` file in the **root** of the repository.

**CRITICAL:** You MUST use a structured file-writing tool (e.g., `write_file`) to create this file. **Do not use shell redirections** (`echo`, `printf`, `>`) as they are prone to escaping errors and produce malformed JSON. Ensure the JSON is valid and pretty-printed with 2-space indentation.

```json
{
  "project_name": "[basename]",
  "version": "1.0",
  "git": {
    "base_branch": "main",
    "remote": "origin"
  },
  "models": {
    "premium": "claude-3-5-sonnet",
    "standard": "gpt-4o",
    "utility": "claude-3-haiku"
  },
  "mcp": {
    "ticket_sync_enabled": true,
    "provider": "jira"
  }
}
```

## Behaviour

1. **Detect Existing Config:** If `.scrum-flow/config.json` already exists, ask if the pilot wants to **update** or **reset** it.
2. **Structured Write:** Generate the JSON object based on user input and write it using `write_file`. Verify the file content immediately after writing to ensure no truncation or malformed characters.
3. **Gitignore Update:** Check if `.scrum-flow/` is ignored. If not, append it to `.gitignore`. **CRITICAL:** Use a structured editing tool (like `replace`) or `write_file` to update `.gitignore`. **Do not use shell redirections** (`echo >>`) as they often fail to handle newlines correctly across different operating systems, leading to malformed entries.
4. **Success Message:** Once configured, provide a summary and the command to start the first feature: `scrumflow start "feature description"`.
