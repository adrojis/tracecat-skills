# tracecat-repo-sync

## Purpose
Synchronize local Tracecat MCP server code and Claude Code skills to their GitHub repositories.

## Activation Triggers
- User explicitly asks to sync changes to GitHub
- User wants to push MCP server updates
- User wants to push skill updates
- User asks about repository structure

## Files
| File | Description |
|------|-------------|
| `SKILL.md` | Main skill — repo mappings, sync procedure, safety rules |
| `README.md` | This file — metadata and overview |
| `COMMON_MISTAKES.md` | Wrong vs right patterns for repo sync |
| `EXAMPLES.md` | Sync procedure examples |

## Dependencies
- GitHub MCP tools (mcp__my-github__*)
- Git CLI

## Success Criteria
- No credentials (.env, secrets) are pushed
- Changes reviewed before push
- Descriptive commit messages
- Correct repository targeted (MCP → tracecat-mcp-community, Skills → tracecat-skills)
- dist/ and node_modules/ excluded
