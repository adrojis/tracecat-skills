# Examples — Validation & Debug

## Example 1: Full Validation Workflow

```
# 1. Validate before deployment
tracecat_validate_workflow({ workflow_id: "wf_xxx" })
# → {
#     "valid": false,
#     "errors": [
#       "Trigger is not connected to any action",
#       "Action \"Score Risk\" has unclosed expression(s): 3 opening vs 2 closing"
#     ],
#     "warnings": [
#       "Action \"Log Event\" is orphaned (no edges)"
#     ]
#   }

# 2. Fix: autofix first
tracecat_autofix_workflow({ workflow_id: "wf_xxx", dry_run: true })
# → { fixes: ["Connect trigger to first action", "Connect orphan 'Log Event'", "Closed 1 unclosed expression"] }

# 3. Apply fixes
tracecat_autofix_workflow({ workflow_id: "wf_xxx" })

# 4. Re-validate
tracecat_validate_workflow({ workflow_id: "wf_xxx" })
# → { "valid": true, "errors": [], "warnings": [] }

# 5. Deploy
tracecat_deploy_workflow({ workflow_id: "wf_xxx" })
```

---

## Example 2: Debug a Failed Execution

```
# 1. Find the failed execution
tracecat_list_executions({ workflow_id: "wf_xxx", limit: 3 })
# → Shows executions with statuses

# 2. Get compact details
tracecat_get_execution_compact({ execution_id: "exec_failed" })
# → Output (simplified):
# {
#   "actions": [
#     { "title": "Parse Input", "status": "COMPLETED", "result": {...} },
#     { "title": "VT Lookup", "status": "FAILED", "error": "HTTP 429: Rate limit exceeded" },
#     { "title": "Score", "status": "SKIPPED" }
#   ]
# }

# 3. The issue: VT API rate limit
# Fix: add retry policy
tracecat_update_action({
  action_id: "act_vt",
  workflow_id: "wf_xxx",
  control_flow: {
    retry_policy: {
      max_attempts: 3,
      timeout: 60
    },
    start_delay: 2
  }
})

# 4. Re-run
tracecat_run_workflow({ workflow_id: "wf_xxx", payload: { ip: "8.8.8.8" } })
```

---

## Example 3: Debug Expression Error

**Error message:** `Action "Create Case" failed: expression error: unknown slug 'enrich_ip'`

```
# 1. List actions to check slugs
tracecat_list_actions({ workflow_id: "wf_xxx" })
# → [
#     { title: "Enrich IP", ref: "enrich_ip_address", ... },
#     { title: "Create Case", ref: "create_case", ... }
#   ]

# 2. The ref is "enrich_ip_address", not "enrich_ip"
# Fix the expression
tracecat_update_action({
  action_id: "act_case",
  workflow_id: "wf_xxx",
  inputs: "case_title: Alert ${{ ACTIONS.enrich_ip_address.result.ip }}\nstatus: new\npriority: high"
})
```

---

## Example 4: Debug Secret Access Error

**Error message:** `Secret 'virus_total' not found`

```
# 1. Search for the secret
tracecat_search_secrets({ name: "virus" })
# → [{ name: "virustotal", id: "uuid-xxx" }]

# 2. The name is "virustotal" (no underscore), not "virus_total"
# Fix the expression in the action:
# Wrong: ${{ SECRETS.virus_total.API_KEY }}
# Right: ${{ SECRETS.virustotal.API_KEY }}

tracecat_update_action({
  action_id: "act_xxx",
  workflow_id: "wf_xxx",
  inputs: "url: https://api.vt.com/v3/...\nmethod: GET\nheaders:\n  x-apikey: ${{ SECRETS.virustotal.API_KEY }}"
})
```

---

## Example 5: Debug Graph Connectivity Issues

```
# 1. Get the graph
tracecat_get_graph({ workflow_id: "wf_xxx" })
# → {
#     "trigger": { "id": "trig_xxx" },
#     "edges": [
#       { "source_id": "trig_xxx", "target_id": "act_1" },
#       { "source_id": "act_1", "target_id": "act_2" }
#     ],
#     "nodes": [
#       { "id": "act_1", ... },
#       { "id": "act_2", ... },
#       { "id": "act_3", ... }  // orphan!
#     ]
#   }

# 2. act_3 is not in any edge → orphan
# Fix: connect it
tracecat_add_edges({
  workflow_id: "wf_xxx",
  edges: [
    { source_id: "act_2", source_type: "udf", target_id: "act_3" }
  ]
})

# 3. Position it
tracecat_move_nodes({
  workflow_id: "wf_xxx",
  positions: [{ action_id: "act_3", x: 500, y: 620 }]
})
```
