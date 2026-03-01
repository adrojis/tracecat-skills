# tracecat-repo-sync

Claude Code skill for synchronizing local Tracecat project files to their GitHub repositories.

## What it does

Automatically detects when local MCP server code or Claude Code skills have been modified and offers to push changes to the corresponding GitHub repos:

- `mcp_server/src/*` -> [adrojis/tracecat-mcp](https://github.com/adrojis/tracecat-mcp)
- `~/.claude/skills/tracecat-*` -> [adrojis/tracecat-skills](https://github.com/adrojis/tracecat-skills)

## Triggers

- Modification of `.ts` files in `mcp_server/src/`
- Modification of skill files in `~/.claude/skills/tracecat-*/`
- Discovery of new API quirks or workflow patterns
- Explicit user request

## Safety

- Always asks for confirmation before pushing
- Never syncs `.env`, credentials, or compiled output
- One commit per logical change with descriptive English messages
