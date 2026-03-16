---
name: tracecat-updates-monitor
description: Monitor Tracecat GitHub releases, documentation changes, API updates, and MCP developments. Run at session start or on-demand to detect new versions, breaking changes, and features that may impact our MCP server or skills.
---

# Tracecat Updates Monitor

You monitor the Tracecat ecosystem for changes that could impact our local MCP server and skills. Run this check systematically and report findings clearly.

## When to Run

- **Session start** — when user begins working on MCP/skills
- **On-demand** — when user invokes this skill explicitly
- **Periodically** — user can use `/loop` to schedule recurring checks

## Check Procedure

### Step 1: Get Current Known State

Search for a memory file named `tracecat-versions.md` in the user's Claude memory directory to know the last checked version and state. Use Grep to find it if the path is unknown.

If the file doesn't exist, create it after the first check.

### Step 2: Check GitHub Releases (CRITICAL)

Run via Bash:
```bash
gh api repos/TracecatHQ/tracecat/releases --jq '.[0:10] | .[] | "## " + .tag_name + " (" + .published_at + ")\n" + .body + "\n---"'
```

Analyze each release for:
- **Version number** — compare with last known version
- **Breaking changes** — anything that affects our API calls, tool definitions, or data formats
- **New features** — new API endpoints, MCP changes, new action types
- **Deprecations** — features we depend on being removed

### Step 3: Check Recent Commits

```bash
gh api repos/TracecatHQ/tracecat/commits --jq '.[0:20] | .[] | .sha[0:7] + " " + .commit.message' | head -30
```

Look for commits mentioning:
- `mcp` — MCP server changes
- `api` — API endpoint changes
- `breaking` — breaking changes
- `registry` — action registry changes
- `table` / `row` / `inventory` — data model changes
- `secret` — secrets management changes
- `agent` — agent/AI features

### Step 4: Check Documentation Updates

Use `WebFetch` on key doc pages:

1. **Changelog/What's New**: `https://docs.tracecat.com/`
2. **API Reference**: `https://docs.tracecat.com/api-reference`
3. **Integrations**: `https://docs.tracecat.com/integrations`
4. **MCP docs** (if they exist): search for MCP-related pages

Also fetch `https://docs.tracecat.com/llms.txt` for the full doc index — compare with known structure.

### Step 5: Check for Official MCP Server

Search for an official Tracecat MCP package:
```bash
gh api search/repositories --jq '.items[] | .full_name + " - " + .description' -f q="tracecat mcp org:TracecatHQ"
```

Also check:
- `https://www.npmjs.com/search?q=tracecat-mcp-community`
- `https://pypi.org/search/?q=tracecat+mcp`
- GitHub MCP servers registry for Tracecat entries

### Step 6: Compare and Report

Generate a report with these sections:

```
## Tracecat Updates Report — [DATE]

### Version Status
- Last checked: [version]
- Current latest: [version]
- New releases: [count] since last check

### Breaking Changes (ACTION REQUIRED)
- [list any breaking changes that affect our MCP server or skills]

### New Features (OPPORTUNITIES)
- [new features we could leverage or need to support]

### MCP Competition Watch
- Official Tracecat MCP server: [not found / found at URL]
- MCP route status: [current state of /mcp endpoint]

### API Endpoint Watch
- New API endpoints: [list any new endpoints]
- Changed endpoints: [list any changed endpoints we use]

### Recommendations
- [specific actions to take: update tools, modify API calls, etc.]
```

### Step 7: Update Memory

After the check, update the memory file `tracecat-versions.md` with:
- Current latest version
- Date of last check
- Summary of key findings
- Any action items

## Impact Assessment Rules

### HIGH impact (flag immediately):
- API endpoint URL/method changes we use
- Authentication changes
- Data format changes (e.g., tables → row inventories)
- New official MCP server released
- Action type renames/removals

### MEDIUM impact (note for next session):
- New action types available
- New API endpoints we could use
- New integrations available
- Documentation structure changes

### LOW impact (log only):
- UI-only changes
- Bug fixes in unrelated areas
- Helm chart updates
- Test/CI changes

## Our MCP Server Dependencies

These are the API patterns our server uses — monitor for changes:

| Feature | Endpoint Pattern | Risk |
|---------|-----------------|------|
| Workflows | `GET/POST /api/workflows` | MEDIUM |
| Actions | `GET/POST /api/actions` | HIGH |
| Executions | `GET /api/workflow-executions` | MEDIUM |
| Cases | `GET/POST /api/cases` | MEDIUM |
| Secrets | `GET/POST /api/secrets` | MEDIUM |
| Tables | `GET/POST /api/tables` | **HIGH** (row inventories migration!) |
| Schedules | `GET/POST /api/schedules` | LOW |
| Graph API | `PATCH /api/workflows/{id}/graph` | HIGH |

## Reference Files
- [Common Mistakes](./COMMON_MISTAKES.md)
- [Examples](./EXAMPLES.md)
- [README](./README.md)
