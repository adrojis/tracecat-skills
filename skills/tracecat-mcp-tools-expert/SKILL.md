---
name: tracecat-mcp-tools-expert
description: Expert guide for using Tracecat MCP tools and API effectively. Use when interacting with Tracecat, managing workflows, creating cases, querying executions, configuring schedules, or using any tracecat MCP tool.
---

# Tracecat MCP Tools Expert

## Quick Reference: Which Tool to Use

| Goal | Tool(s) | Order |
|------|---------|-------|
| See all workflows | `list_workflows` | Single call |
| Inspect a workflow | `get_workflow` | Single call |
| Create & deploy a workflow | `create_workflow` -> `create_action` (xN) -> `commit_workflow` | Sequential |
| Run a workflow | `trigger_workflow` | Requires committed + online workflow |
| Check run results | `list_executions` -> `get_execution` | Sequential |
| Manage incidents | `create_case` -> `update_case` -> `create_case_comment` | As needed |
| Set up scheduling | `create_schedule` | Requires committed workflow |
| Manage secrets | `list_secrets` / `create_secret` / `delete_secret` | As needed |

## Critical Rules

1. **Workflow lifecycle**: Save -> Commit -> Activate. A workflow MUST be committed before it can be triggered or scheduled.
2. **Always `list_workflows` first** before creating, to avoid duplicates.
3. **Workspace ID** is required on nearly all API calls as a query parameter.
4. **Workflow IDs** follow pattern `wf-XXXXXXXXXX`.
5. **Action IDs** follow pattern `act-XXXXXXXXXX`.

## API Authentication

- Cookie-based session auth (`fastapiusersauth`)
- Login: `POST /auth/login` (form-urlencoded: username + password)
- Service key (`x-tracecat-service-key`) does NOT work on user endpoints

## Workflow Deployment Sequence

```
1. create_workflow (title, description)
   -> Returns workflow_id

2. create_action (workflow_id, type, title) x N
   -> Returns action IDs and refs

3. update_action (action_id, workflow_id, inputs=YAML_STRING) x N
   -> Configure each action's inputs

4. Link nodes via Graph API:
   PATCH /workflows/{id}/graph?workspace_id=...
   Body: { base_version, operations: [{ type: "add_edge", payload: {...} }] }
   -> Trigger edge: source_type="trigger", source_id="trigger-{internal_uuid}"
   -> Action edges: source_type="udf"

5. Position nodes visually:
   operations: [{ type: "move_nodes", payload: { positions: [{action_id, x, y}] } }]
   operations: [{ type: "update_trigger_position", payload: { x, y } }]

6. deploy_workflow (commit) -> validates and creates definition

7. Set workflow status to "online" (PATCH /workflows/{id})

8. Set webhook to "online" (PATCH /workflows/{id}/webhook)

9. Trigger via webhook URL (NOT /workflows/{id}/execute which returns 404):
   POST /api/webhooks/{wf_id}/{webhook_secret}
```

**IMPORTANT**: The MCP tool `tracecat_run_workflow` calls `/workflows/{id}/execute` which
does NOT exist in Tracecat v1.0.0-beta.9. Use the webhook URL to trigger workflows instead.

## Expression Syntax Quick Reference

```
${{ TRIGGER.field }}                    # Webhook input
${{ ACTIONS.action_slug.result }}       # Action output
${{ ACTIONS.ref.result.data[0].name }}  # Nested JSONPath
${{ SECRETS.name.KEY }}                 # Secret value
${{ FN.now() }}                         # Function call
${{ "default" if x == None else x }}    # Inline ternary
${{ ACTIONS.x.result || "fallback" }}   # Default value
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Triggering uncommitted workflow | `deploy_workflow` first |
| Using `tracecat_run_workflow` MCP tool | Returns 404. Trigger via webhook URL instead |
| Scheduling uncommitted workflow | Commit first, then schedule |
| Nesting `${{ }}` inside `${{ }}` | Use single `${{ FN.func1(FN.func2()) }}` |
| Using `body` instead of `payload` | HTTP actions use `payload` for request body |
| Missing `[*]` in JSONPath for lists | `result.data[*].name` not `result.data.name` |
| Dots in field names | Use quotes: `result."kibana.alert.field"` |
| `run_python` with top-level `return` | Code MUST be inside a function (`def process():`) |
| Booleans in expressions as `true`/`false` | Use Python-style: `True`/`False` |
| Actions not linked (no edges) | Use Graph API `add_edge` operations after creating actions |
| Nodes stacked at (0,0) in UI | Use `move_nodes` with `{action_id, x, y}` positions |
| `tracecat_update_workflow` returns empty JSON | Known bug in MCP tool; use direct API call instead |