# Examples — MCP Tools

## Example 1: Create a Complete Workflow (End-to-End)

```
# 1. Create workflow
tracecat_create_workflow({ title: "IP Enrichment", description: "Enrich IPs via VT and AbuseIPDB" })
# → Returns: { id: "wf_xxx", ... }

# 2. Create actions
tracecat_create_action({ workflow_id: "wf_xxx", type: "core.transform.reshape", title: "Parse Input" })
# → Returns: { id: "act_1", ... }
tracecat_create_action({ workflow_id: "wf_xxx", type: "core.http_request", title: "VT Lookup" })
# → Returns: { id: "act_2", ... }
tracecat_create_action({ workflow_id: "wf_xxx", type: "core.transform.reshape", title: "Score Result" })
# → Returns: { id: "act_3", ... }

# 3. Configure inputs (YAML strings!)
tracecat_update_action({
  action_id: "act_1", workflow_id: "wf_xxx",
  inputs: "value:\n  ip: ${{ TRIGGER.data.ip }}"
})

tracecat_update_action({
  action_id: "act_2", workflow_id: "wf_xxx",
  inputs: "url: https://www.virustotal.com/api/v3/ip_addresses/${{ ACTIONS.parse_input.result.ip }}\nmethod: GET\nheaders:\n  x-apikey: ${{ SECRETS.virustotal.API_KEY }}"
})

tracecat_update_action({
  action_id: "act_3", workflow_id: "wf_xxx",
  inputs: "value:\n  ip: ${{ ACTIONS.parse_input.result.ip }}\n  malicious: ${{ ACTIONS.vt_lookup.result.data.attributes.last_analysis_stats.malicious }}\n  is_threat: ${{ ACTIONS.vt_lookup.result.data.attributes.last_analysis_stats.malicious > 5 }}"
})

# 4. Get graph to find trigger ID
tracecat_get_graph({ workflow_id: "wf_xxx" })
# → Returns: { trigger: { id: "trigger_xxx" }, edges: [], ... }

# 5. Connect edges
tracecat_add_edges({
  workflow_id: "wf_xxx",
  edges: [
    { source_id: "trigger_xxx", source_type: "trigger", target_id: "act_1" },
    { source_id: "act_1", source_type: "udf", target_id: "act_2" },
    { source_id: "act_2", source_type: "udf", target_id: "act_3" }
  ]
})

# 6. Position nodes
tracecat_update_trigger_position({ workflow_id: "wf_xxx", x: 500, y: 0 })
tracecat_move_nodes({
  workflow_id: "wf_xxx",
  positions: [
    { action_id: "act_1", x: 500, y: 300 },
    { action_id: "act_2", x: 500, y: 460 },
    { action_id: "act_3", x: 500, y: 620 }
  ]
})

# 7. Validate
tracecat_validate_workflow({ workflow_id: "wf_xxx" })

# 8. Deploy
tracecat_deploy_workflow({ workflow_id: "wf_xxx" })
```

---

## Example 2: Investigate a Failed Execution

```
# 1. List recent executions
tracecat_list_executions({ workflow_id: "wf_xxx", limit: 5 })

# 2. Get compact details of the failed one
tracecat_get_execution_compact({ execution_id: "exec_failed" })
# → Shows each action's status, inputs, results, and errors

# 3. Find the failing action (status: "FAILED")
# → e.g., "VT Lookup" failed with "HTTP 403: Forbidden"

# 4. Check the action configuration
tracecat_get_action({ action_id: "act_vt", workflow_id: "wf_xxx" })

# 5. Fix: update the secret or inputs
tracecat_update_action({
  action_id: "act_vt",
  workflow_id: "wf_xxx",
  inputs: "url: https://www.virustotal.com/api/v3/ip_addresses/${{ ACTIONS.parse.result.ip }}\nmethod: GET\nheaders:\n  x-apikey: ${{ SECRETS.virustotal.API_KEY }}"
})

# 6. Re-run
tracecat_run_workflow({ workflow_id: "wf_xxx", payload: { ip: "8.8.8.8" } })
```

---

## Example 3: Set Up Tables for IOC Tracking

```
# 1. Create table
tracecat_create_table({ name: "ioc_tracker", description: "Track enriched IOCs" })
# → Returns: { id: "tbl_xxx" }

# 2. Add columns
tracecat_create_column({ table_id: "tbl_xxx", name: "ip_address", type: "TEXT" })
tracecat_create_column({ table_id: "tbl_xxx", name: "vt_score", type: "INTEGER" })
tracecat_create_column({ table_id: "tbl_xxx", name: "abuse_score", type: "INTEGER" })
tracecat_create_column({ table_id: "tbl_xxx", name: "verdict", type: "TEXT" })
tracecat_create_column({ table_id: "tbl_xxx", name: "first_seen", type: "TIMESTAMPTZ" })

# 3. Insert rows
tracecat_batch_insert_rows({
  table_id: "tbl_xxx",
  rows: [
    { ip_address: "1.2.3.4", vt_score: 45, abuse_score: 90, verdict: "malicious" },
    { ip_address: "5.6.7.8", vt_score: 2, abuse_score: 5, verdict: "clean" }
  ]
})

# 4. Query rows
tracecat_list_rows({ table_id: "tbl_xxx", limit: 50 })
```

---

## Example 4: Schedule a Workflow

```
# Create hourly schedule
tracecat_create_schedule({
  workflow_id: "wf_scan",
  every: "1h",
  inputs: { targets: ["10.0.0.0/24"], scan_type: "vulnerability" }
})

# Or cron-based (daily at 6 AM)
tracecat_create_schedule({
  workflow_id: "wf_report",
  cron: "0 6 * * *"
})

# Pause a schedule
tracecat_update_schedule({ schedule_id: "sched_xxx", status: "offline" })
```

---

## Example 5: Autofix and Validate Workflow

```
# 1. Dry run to see what would be fixed
tracecat_autofix_workflow({ workflow_id: "wf_xxx", dry_run: true })
# → { fixes: ["Connect orphan 'Log Event' after last node"], unfixable: [] }

# 2. Apply fixes
tracecat_autofix_workflow({ workflow_id: "wf_xxx" })
# → { fixes_applied: 2, message: "2 fix(es) applied successfully" }

# 3. Validate
tracecat_validate_workflow({ workflow_id: "wf_xxx" })
# → { valid: true, errors: [], warnings: [] }

# 4. Deploy
tracecat_deploy_workflow({ workflow_id: "wf_xxx" })
```
