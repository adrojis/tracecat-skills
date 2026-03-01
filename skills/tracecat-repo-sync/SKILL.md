---
name: tracecat-repo-sync
description: Synchronize local Tracecat MCP server code and Claude Code skills to their GitHub repositories. Use when modifying MCP server TypeScript files, updating skills, discovering new API quirks, or when the user asks to sync changes.
---

# Tracecat Repo Sync

Synchronize local changes to the upstream GitHub repositories:
- **MCP Server code** -> `adrojis/tracecat-mcp` (branch: `main`)
- **Skills** -> `adrojis/tracecat-skills` (branch: `main`)

## When to Trigger

Proactively suggest syncing when:
1. A `.ts` file in `mcp_server/src/` has been modified
2. A `SKILL.md` in `~/.claude/skills/tracecat-*` has been modified
3. A new API quirk or workflow pattern has been discovered and documented
4. The user explicitly asks to sync (`/repo-sync` or "sync to GitHub")

## File Mappings

| Local Path | GitHub Repo | Remote Path |
|------------|-------------|-------------|
| `mcp_server/src/*` | `adrojis/tracecat-mcp` | `src/*` |
| `~/.claude/skills/tracecat-*/SKILL.md` | `adrojis/tracecat-skills` | `skills/tracecat-*/SKILL.md` |
| `~/.claude/skills/tracecat-*/README.md` | `adrojis/tracecat-skills` | `skills/tracecat-*/README.md` |

## Sync Procedure

### Step 1: Identify Changed Files

Compare local files against their GitHub versions:
- Read the local file content
- Fetch the remote file via `mcp__my-github__get_file_contents` to get its SHA and content
- If they differ, the file needs syncing

### Step 2: Confirm with User

Before pushing, show the user:
- Which files will be updated
- Which repo/branch they'll go to
- A proposed commit message

**Always ask for confirmation before pushing.** Never push without explicit user approval.

### Step 3: Push Changes

For each file to sync:

1. Read the local file content
2. Get the current SHA from GitHub via `mcp__my-github__get_file_contents`
3. Push via `mcp__my-github__create_or_update_file` with:
   - `owner`: `adrojis`
   - `repo`: target repo name
   - `path`: mapped remote path
   - `content`: local file content
   - `message`: descriptive commit message in English
   - `branch`: `main`
   - `sha`: current file SHA (required for updates, omit for new files)

### Step 4: Confirm Success

Report which files were synced and provide links to the commits.

## Sync Rules

1. **Always confirm** before pushing — never auto-push
2. **Never sync** `.env`, credentials, or secrets files
3. **Never sync** `mcp_server/dist/` (compiled output) or `node_modules/`
4. **Commit messages** must be in English, descriptive, and follow conventional style:
   - `feat: add schedule management tools`
   - `fix: correct workspace_id query param handling`
   - `docs: update API quirks in README`
   - `refactor: simplify client authentication flow`
5. **One commit per logical change** — don't bundle unrelated changes
6. **For new API quirks**: update the README.md of `adrojis/tracecat-mcp` (add to API Quirks section)
7. **For new workflow patterns**: update the README.md of `adrojis/tracecat-skills` (add to patterns section)

## MCP Server Sync — Full File List

Files under `mcp_server/src/` that map to `src/` in `adrojis/tracecat-mcp`:

```
src/index.ts
src/server.ts
src/client.ts
src/types.ts
src/tools/*.ts
```

Also sync root config files if changed:
```
package.json
tsconfig.json
README.md
```

## Skills Sync — Full Skill List

Skills under `~/.claude/skills/` that map to `skills/` in `adrojis/tracecat-skills`:

```
skills/tracecat-action-configuration/
skills/tracecat-case-management/
skills/tracecat-mcp-tools-expert/
skills/tracecat-secrets-integrations/
skills/tracecat-workflow-patterns/
skills/tracecat-repo-sync/
```

## Example Usage

**User modifies `mcp_server/src/tools/workflows.ts`:**

> "I notice you've modified `mcp_server/src/tools/workflows.ts`. Would you like me to sync this to `adrojis/tracecat-mcp`?"

Then if confirmed:
1. Read local `mcp_server/src/tools/workflows.ts`
2. Get SHA from `adrojis/tracecat-mcp` for `src/tools/workflows.ts`
3. Push with message like `feat: add workflow export tool`

**User discovers a new API quirk:**

> "I've documented a new API quirk. Would you like me to update the README in `adrojis/tracecat-mcp` with this finding?"

**Bulk sync:**

> User: "sync everything to GitHub"

1. List all local files in `mcp_server/src/` and `~/.claude/skills/tracecat-*/`
2. Compare each against GitHub
3. Show diff summary
4. Ask for confirmation
5. Push changed files with appropriate commit messages
