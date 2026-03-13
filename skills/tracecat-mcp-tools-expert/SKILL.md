---
name: tracecat-mcp-tools-expert
description: Activate when users interact with Tracecat MCP tools, create workflows, manage cases, or need help with Tracecat API operations
---

# Tracecat MCP Tools Expert

You are an expert at using Tracecat MCP tools to manage workflows, cases, executions, and secrets in a Tracecat SOAR platform.

## Critical Rule: Always Use Native Action Types

When creating actions with `tracecat_create_action`, ALWAYS use native/official action types before falling back to generic ones:
- Cases → `core.cases.create_case`, `core.cases.update_case`, `core.cases.create_comment` (NEVER reshape)
- Integrations → `tools.virustotal.*`, `tools.crowdstrike.*`, `tools.splunk.*`, `tools.okta.*`, `tools.microsoft_defender.*`, `tools.sharepoint.*` (NEVER core.http_request when native exists)
- `core.http_request` → ONLY when no native integration exists
- `core.transform.reshape` → ONLY for data transformation, never for CRUD
- `core.script.run_python` → ONLY when logic can't be expressed otherwise

## Available Tools (51 total)

### System (1)
- `tracecat_health_check` — Always start here to verify Tracecat is running

### Workflows (8)
- `tracecat_list_workflows` — List all workflows
- `tracecat_create_workflow` — Create a new workflow (title + description)
- `tracecat_get_workflow` — Get workflow details by ID
- `tracecat_update_workflow` — Update title, description, or status
- `tracecat_deploy_workflow` — Commit/deploy a workflow to make it active
- `tracecat_export_workflow` — Export workflow YAML definition
- `tracecat_delete_workflow` — Delete a workflow permanently
- `tracecat_validate_workflow` — Validate workflow without deploying (checks actions, inputs, expressions, graph connectivity)

### Actions (5)
- `tracecat_list_actions` — List all actions in a workflow
- `tracecat_create_action` — Create a new action (type + title, requires workflow_id)
- `tracecat_get_action` — Get action details by ID
- `tracecat_update_action` — Update title, description, inputs, control_flow
- `tracecat_delete_action` — Delete an action

### Executions (5)
- `tracecat_run_workflow` — Execute a workflow with optional payload
- `tracecat_list_executions` — List executions (optionally filter by workflow)
- `tracecat_get_execution` — Get full execution details with events
- `tracecat_get_execution_compact` — Get compact execution (action-level status, inputs, results, errors)
- `tracecat_cancel_execution` — Cancel a running execution

### Cases (7)
- `tracecat_list_cases` — List cases (optionally filter by status)
- `tracecat_create_case` — Create a case linked to a workflow
- `tracecat_get_case` — Get a specific case by ID
- `tracecat_update_case` — Update case status, priority, etc.
- `tracecat_delete_case` — Delete a case permanently
- `tracecat_add_comment` — Add a comment to a case
- `tracecat_list_comments` — List all comments on a case

### Secrets (5)
- `tracecat_search_secrets` — Search secrets by name
- `tracecat_create_secret` — Create a new secret with key-value pairs
- `tracecat_get_secret` — Get secret metadata by name
- `tracecat_update_secret` — Update secret keys or description
- `tracecat_delete_secret` — Delete a secret permanently

### Tables (5)
- `tracecat_list_tables` — List all tables
- `tracecat_create_table` — Create a new table
- `tracecat_get_table` — Get table details by ID (includes columns)
- `tracecat_update_table` — Update table name or description
- `tracecat_delete_table` — Delete a table

### Table Columns (2)
- `tracecat_create_column` — Create a column (types: TEXT, INTEGER, NUMERIC, DATE, BOOLEAN, TIMESTAMP, TIMESTAMPTZ, JSONB, UUID, SELECT, MULTI_SELECT)
- `tracecat_delete_column` — Delete a column

### Table Rows (6)
- `tracecat_list_rows` — List rows with optional pagination
- `tracecat_get_row` — Get a specific row by ID
- `tracecat_insert_row` — Insert a new row
- `tracecat_update_row` — Update an existing row
- `tracecat_delete_row` — Delete a row
- `tracecat_batch_insert_rows` — Insert multiple rows at once

### Schedules (5)
- `tracecat_list_schedules` — List schedules (optionally filter by workflow)
- `tracecat_create_schedule` — Create a cron or interval schedule for a workflow
- `tracecat_get_schedule` — Get schedule details
- `tracecat_update_schedule` — Update cron, interval, status, or inputs
- `tracecat_delete_schedule` — Delete a schedule

### Webhooks (1)
- `tracecat_create_webhook_key` — Generate/rotate a webhook API key

### Graph (5)
- `tracecat_get_graph` — Get workflow graph (nodes, edges, positions, version)
- `tracecat_add_edges` — Add connections between actions (source_type: 'trigger' or 'udf')
- `tracecat_delete_edges` — Remove connections between actions
- `tracecat_move_nodes` — Reposition action nodes (recommended: trigger at x=500,y=0, 160px vertical spacing)
- `tracecat_update_trigger_position` — Reposition the trigger node

### Documentation (1)
- `tracecat_docs` — Get inline documentation (topics: action_types, expressions, functions, control_flow, common_mistakes)

## Recommended Operation Order

### Creating a new workflow from scratch
1. `tracecat_health_check` — Verify connectivity
2. `tracecat_list_workflows` — Check existing workflows
3. `tracecat_create_workflow` — Create the workflow
4. `tracecat_create_action` — Add actions one by one
5. `tracecat_update_action` — Configure inputs and control flow for each action
6. `tracecat_add_edges` — Connect trigger → actions → actions (use source_type 'trigger' for first edge)
7. `tracecat_move_nodes` — Position all nodes (trigger at 500,0; first action at 500,300; 160px spacing)
8. `tracecat_validate_workflow` — Check for errors before deploying
9. `tracecat_deploy_workflow` — Deploy to make it active
10. `tracecat_run_workflow` — Test execution
11. `tracecat_get_execution_compact` — Check results

### Investigating an incident
1. `tracecat_list_cases` with status filter
2. `tracecat_get_case` — Get full case details
3. `tracecat_update_case` — Set to `in_progress`
4. `tracecat_add_comment` — Document findings
5. `tracecat_run_workflow` — Trigger remediation workflows
6. `tracecat_update_case` — Set to `resolved`

### Checking execution results
1. `tracecat_list_executions` — Find the execution
2. `tracecat_get_execution_compact` — Quick triage (action status + errors)
3. `tracecat_get_execution` — Deep investigation (full event history)

### Setting up a scheduled workflow
1. Create and deploy the workflow first
2. `tracecat_create_schedule` — Set cron or interval
3. `tracecat_list_schedules` — Verify it's active

### Working with tables
1. `tracecat_create_table` — Create the table
2. `tracecat_insert_row` / `tracecat_batch_insert_rows` — Add data
3. `tracecat_list_rows` — Query data
4. `tracecat_update_row` — Modify entries

## Graph API — Node Positioning

Use the **Graph tools** (`tracecat_get_graph`, `tracecat_add_edges`, `tracecat_delete_edges`, `tracecat_move_nodes`, `tracecat_update_trigger_position`) to manage node connections and positions. When creating actions, nodes default to `(0, 0)` and overlap. **You MUST reposition nodes** after creating/wiring them.

### API Endpoint
```
PATCH /api/workflows/{workflow_id}/graph?workspace_id={workspace_id}
```
Requires **cookie-based auth** (login via `POST /auth/login` first), NOT service key.

### move_nodes Operation (validated format)
```json
{
  "base_version": <current_graph_version>,
  "operations": [
    {
      "type": "move_nodes",
      "payload": {
        "positions": [
          {"action_id": "<uuid>", "x": 250, "y": 200},
          {"action_id": "<uuid>", "x": 250, "y": 400}
        ]
      }
    }
  ]
}
```

**CRITICAL format rules:**
- Field is `action_id` (NOT `node_id`, NOT `id`)
- Coordinates `x` and `y` are **flat** at root level (NOT nested in `position: {x, y}`)
- `positions` is a **list** (NOT a dict)
- `base_version` must match the current graph version (get it with GET on the graph endpoint)

### Other Graph Operations
| Operation | Payload fields |
|---|---|
| `add_edge` | `source_id`, `source_type` ("trigger" or "udf"), `target_id` |
| `delete_edge` | `source_id`, `source_type`, `target_id` |
| `update_node` | `action_id`, plus fields to update |
| `move_nodes` | `positions`: list of `{action_id, x, y}` |
| `update_trigger_position` | `x`, `y` |
| `update_viewport` | `x`, `y`, `zoom` |

### Node Layout Best Practices

**Spacing rules (validated in Tracecat UI):**
- **Vertical spacing between levels: 200px minimum** — Nodes are ~100px tall. Less than 200px causes overlap.
- **Horizontal spacing between siblings: 350px minimum** — Nodes are ~250px wide. Less than 350px causes overlap for case/action nodes with long titles.
- **Branch offset: 250-300px** from center axis — When a node branches (e.g. if/else), offset children left and right from parent's x.

**Layout algorithm for common patterns:**

1. **Linear chain** (A → B → C): Same x, increment y by 200
   ```
   A (250, 0) → B (250, 200) → C (250, 400)
   ```

2. **Binary branch** (A → B | C): Parent centered, children offset ±300
   ```
   A (250, 600)
   B (0, 850)    C (600, 850)
   ```

3. **Triple fan-out** (A → B | C | D): Parent centered, children at -350, 0, +350
   ```
   A (0, 1000)
   B (-350, 1250)  C (0, 1250)  D (350, 1250)
   ```

4. **Full workflow example** (trigger → chain → branch → fan-out):
   ```
   Trigger          (250, 0)
   Step 1           (250, 250)
   Step 2           (250, 450)
   Decision         (250, 650)
   Left branch      (0, 880)        Right branch  (600, 880)
   Sub-step         (0, 1100)
   Case A (-350, 1350)  Case B (0, 1350)  Case C (350, 1350)
   ```

**Key principles:**
- Keep the main chain centered on x=250 (aligns with default trigger position)
- Use consistent vertical gaps (200-230px for tight layouts, 250px for readability)
- For branches, offset symmetrically from the parent's x coordinate
- For fan-outs of 3+ nodes, calculate: `parent_x + (i - (n-1)/2) * 350` for each child i

## Tips
- Workflow IDs are UUIDs like `wf:abc123...`
- Always deploy a workflow after making changes
- **Always reposition nodes after creating them** — MCP-created nodes default to (0,0)
- Executions are asynchronous; use `tracecat_get_execution_compact` for quick status checks
- The workspace ID is auto-detected on server start
- Use `tracecat_create_action` with `type` matching the action registry (e.g. `core.transform.reshape`, `core.http_request`, `core.script.run_python`)
- `tracecat_update_action` accepts `inputs` as a **YAML string** (not JSON object) and `control_flow` (run_if, for_each, retry_policy)
- Actions require `workflow_id` on every operation (get, update, delete)
- Tables require columns to be created before inserting rows (use `tracecat_create_column`)
- Batch insert rows expects flat objects (keys = column names), not wrapped in `data`
- Expression operators: use `&&` and `||` (NOT `and`/`or`)
- Use `tracecat_validate_workflow` before deploying to catch common errors early
- Use `tracecat_docs` to get inline reference on action types, expressions, functions, or common mistakes

## Related Skills
- **tracecat-workflow-patterns** — Design patterns and templates for SOAR workflows
- **tracecat-yaml-syntax** — YAML syntax reference for workflow definitions
- **tracecat-integration-expert** — Configure external tool integrations
- **tracecat-case-management** — Case lifecycle and incident tracking
- **tracecat-validation-debug** — Debug failed executions and troubleshoot errors
- **tracecat-code-python** — Write Python scripts for run_python actions

## Reference Files
- [Common Mistakes](./COMMON_MISTAKES.md)
- [Examples](./EXAMPLES.md)
- [README](./README.md)
