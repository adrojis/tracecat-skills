# Common Mistakes — MCP Tools

## 1. Inputs as JSON object instead of YAML string
**Wrong:**
```
tracecat_update_action({
  action_id: "act_xxx",
  workflow_id: "wf_xxx",
  inputs: { url: "https://...", method: "GET" }
})
```
**Right:**
```
tracecat_update_action({
  action_id: "act_xxx",
  workflow_id: "wf_xxx",
  inputs: "url: https://...\nmethod: GET"
})
```
**Why:** The API expects `inputs` as a YAML string. JSON objects cause a 422 validation error.

## 2. Creating actions without edges and positions
**Wrong:**
```
tracecat_create_action(...)
tracecat_create_action(...)
# Done! (actions exist but invisible in UI)
```
**Right:**
```
tracecat_create_action(...)
tracecat_create_action(...)
tracecat_add_edges(...)     # Connect them
tracecat_move_nodes(...)    # Position them
```
**Why:** MCP-created actions default to position (0,0) with no edges. They're invisible/overlapping in the UI.

## 3. First action too close to trigger
**Wrong:**
```
tracecat_move_nodes({
  positions: [{ action_id: "act_xxx", x: 500, y: 200 }]
})
```
**Right:**
```
tracecat_move_nodes({
  positions: [{ action_id: "act_xxx", x: 500, y: 300 }]
})
```
**Why:** The trigger node at y=0 is ~200px tall. First action at y=200 overlaps. Use y >= 300.

## 4. Wrong source_type in edges
**Wrong:**
```
tracecat_add_edges({
  edges: [{ source_id: "trigger_id", source_type: "udf", target_id: "act_xxx" }]
})
```
**Right:**
```
tracecat_add_edges({
  edges: [{ source_id: "trigger_id", source_type: "trigger", target_id: "act_xxx" }]
})
```
**Why:** The trigger node uses `source_type: "trigger"`. Action nodes use `source_type: "udf"`.

## 5. Forgetting workspace_id
**Wrong (raw API):**
```
GET /actions?action_id=act_xxx
```
**Right:**
MCP tools auto-inject `workspace_id`. If using raw API:
```
GET /actions/act_xxx?workspace_id=ws_xxx
```
**Why:** Almost all endpoints require `workspace_id` as a query parameter.

## 6. Using PATCH for action/secret/schedule updates
**Wrong (raw API):**
```
PATCH /actions/{id}
```
**Right:**
```
POST /actions/{id}?workflow_id=wf_xxx
```
**Why:** Tracecat uses POST (not PATCH) for updates on actions, secrets, and schedules. MCP tools handle this automatically.

## 7. Wrong action listing endpoint
**Wrong:**
```
GET /workflows/{id}/actions
```
**Right:**
```
GET /actions?workflow_id={id}
```
**Why:** Actions are listed via `/actions` with `workflow_id` as a query parameter, not nested under workflows.

## 8. Missing dry_run on autofix
**Risky:**
```
tracecat_autofix_workflow({ workflow_id: "wf_xxx" })
```
**Safer first:**
```
tracecat_autofix_workflow({ workflow_id: "wf_xxx", dry_run: true })
# Review planned fixes, then:
tracecat_autofix_workflow({ workflow_id: "wf_xxx" })
```
**Why:** Autofix modifies the graph. Use `dry_run: true` first to review planned changes.
