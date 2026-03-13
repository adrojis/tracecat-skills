# Examples — tracecat-updates-monitor

## Example 1: Standard Session Check

**User:** "Check for Tracecat updates"

**Expected output:**
```
## Tracecat Updates Report — 2026-03-12

### Version Status
- Last checked: 1.0.0-beta.28 (2026-03-09)
- Current latest: 1.0.0-beta.31 (2026-03-12)
- New releases: 3 since last check

### Breaking Changes (ACTION REQUIRED)
- beta.29: "Replace workspace management tables with row inventories" (#2317)
  → Our MCP tools `tracecat_create_table`, `tracecat_list_tables`, etc. may need updating
  → Need to test table endpoints against current API

### New Features (OPPORTUNITIES)
- beta.29: Workflow folder tools, case tag tools — could add to our MCP server
- beta.28: MCP routing through Caddy at /mcp
- beta.28: Chat UI 2.0 with agent presets

### MCP Competition Watch
- Official standalone MCP server: NOT FOUND
- Native /mcp route: Present since beta.28, for internal agent use only

### API Endpoint Watch
- No endpoint URL changes detected
- Registry action deletion prevention added (beta.28)

### Recommendations
1. TEST table/row endpoints — verify our tools still work with row inventories
2. CONSIDER adding workflow folder and tag tools
3. MONITOR /mcp route evolution in future releases
```

## Example 2: Breaking Change Detected

**Scenario:** API endpoint changes

**Expected output:**
```
### Breaking Changes (ACTION REQUIRED)

HIGH IMPACT: Action endpoint changed
- Old: POST /api/actions/{id} (update)
- New: PATCH /api/actions/{id} (update)
- Affects: tracecat_update_action tool in our MCP server
- Action: Update client.ts request method for action updates

HIGH IMPACT: Tables replaced by inventories
- Old: GET/POST /api/tables, /api/tables/{id}/rows
- New: GET/POST /api/inventories, /api/inventories/{id}/rows
- Affects: 5 MCP tools (create_table, list_tables, get_table, delete_table, list_rows...)
- Action: Update all table endpoints in client.ts + rename tools
```

## Example 3: Official MCP Server Found

**Scenario:** Tracecat publishes their own MCP package

**Expected output:**
```
### MCP Competition Watch

CRITICAL: Official Tracecat MCP server found!
- Package: @tracecathq/mcp-server (npm)
- Repository: github.com/TracecatHQ/tracecat-mcp
- Tools count: 35
- Overlap with our server: ~80% (28 tools in common)

Analysis:
- Their tools we DON'T have: agent_chat, preset_manage, folder_create
- Our tools they DON'T have: tracecat_docs, tracecat_tools_documentation, templates
- Their auth: Uses /mcp SSE endpoint (requires Tracecat instance URL)
- Our auth: Direct API with cookie auth

Recommendation:
- Our server still has value (docs, templates, offline reference)
- Consider migrating to their server for core CRUD tools
- Keep our custom tools as an extension
```
