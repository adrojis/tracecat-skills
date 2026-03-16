---
name: tracecat-repo-sync
description: Activate when user asks to sync, push, or publish local Tracecat MCP server code or Claude Code skills to their GitHub repositories
---

# Tracecat Repository Sync

You help synchronize local Tracecat MCP server code and skills to their GitHub repositories.

## Repositories

| Component | Local Path | GitHub Repo |
|-----------|-----------|-------------|
| MCP Server | `<project_root>/mcp_server/` | Owner's `tracecat-mcp-community` repo |
| Skills | `<project_root>/tracecat_skills/` | Owner's `tracecat-skills` repo |

> **Note:** Local paths and GitHub owner are resolved at runtime from the working directory and git config.

## Sync Procedure

### 1. Identify changed files
Compare local files with the GitHub repository to find what has changed.

### 2. Review changes
Show the user what files will be pushed and what content has changed.

### 3. Push to GitHub
Use `mcp__my-github__push_files` or `mcp__my-github__create_or_update_file` to push changes.

## MCP Server Files to Sync

Key files in the MCP server:
```
mcp_server/
  src/
    index.ts          # Entry point
    client.ts         # API client
    tools.ts          # MCP tool definitions
    types.ts          # TypeScript types
  package.json
  tsconfig.json
  CLAUDE.md           # Dev guide
  .env                # DO NOT SYNC (credentials)
```

**Never sync:** `.env`, `node_modules/`, `dist/`, `.env.local`

## Skills Files to Sync

```
tracecat_skills/
  .claude-plugin/
    plugin.json       # Plugin manifest
  skills/
    tracecat-action-configuration/SKILL.md
    tracecat-case-management/SKILL.md
    tracecat-code-python/SKILL.md
    tracecat-integration-expert/SKILL.md
    tracecat-mcp-tools-expert/SKILL.md
    tracecat-secrets-integrations/SKILL.md
    tracecat-validation-debug/SKILL.md
    tracecat-workflow-patterns/SKILL.md
    tracecat-yaml-syntax/SKILL.md
    tracecat-repo-sync/SKILL.md
```

## GitHub MCP Tools

| Tool | Usage |
|------|-------|
| `mcp__my-github__get_file_contents` | Read current file from GitHub |
| `mcp__my-github__create_or_update_file` | Create or update a single file (needs SHA for updates) |
| `mcp__my-github__push_files` | Push multiple files in a single commit |
| `mcp__my-github__list_commits` | Check latest commits |

## Safety Rules

1. **Never push `.env` files** or any file containing credentials
2. **Never push `.mcp.json`** — contains local paths and potentially sensitive env vars
3. **Always show diff** to user before pushing
4. **Use descriptive commit messages** summarizing changes
5. **Push to `main` branch** unless user specifies otherwise
6. **Check for existing files** first to get SHA (required for updates)
7. **Scan for local paths** before pushing — reject any file containing `C:\Users\`, `/home/`, or absolute local paths that would leak user info
8. **Scan for hardcoded credentials** — reject any file containing passwords, API keys, or tokens in plain text

## Related Skills
- **tracecat-mcp-tools-expert** — MCP tools reference

## Reference Files
- [Common Mistakes](./COMMON_MISTAKES.md)
- [Examples](./EXAMPLES.md)
- [README](./README.md)
