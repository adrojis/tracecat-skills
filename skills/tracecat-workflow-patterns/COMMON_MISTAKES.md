# Common Mistakes — Workflow Patterns

## 1. Not connecting edges after creating actions
**Wrong:**
```
tracecat_create_action(...)  # action 1
tracecat_create_action(...)  # action 2
tracecat_create_action(...)  # action 3
# Done! (3 orphan nodes at 0,0)
```
**Right:**
```
tracecat_create_action(...)  # action 1
tracecat_create_action(...)  # action 2
tracecat_create_action(...)  # action 3
tracecat_add_edges(...)      # Connect trigger → 1 → 2 → 3
tracecat_move_nodes(...)     # Position at proper coordinates
```
**Why:** Actions without edges are orphans — invisible in the UI and never executed.

## 2. No error handling for external API calls
**Wrong:**
```
Trigger → VT lookup → Create case
```
**Right:**
```
Trigger → VT lookup ─success─→ Create case
                    ─error──→ Log error + Create case with "enrichment failed"
```
**Why:** External APIs fail (rate limits, timeouts, auth issues). Error paths ensure the workflow continues.

## 3. Sequential when parallel is possible
**Wrong:**
```
Trigger → VT lookup → AbuseIPDB lookup → GreyNoise lookup → Merge
```
**Right:**
```
Trigger ─→ VT lookup ──────→ Merge (join_strategy: all)
        ─→ AbuseIPDB lookup ─→
        ─→ GreyNoise lookup ──→
```
**Why:** Independent enrichment lookups should run in parallel to reduce total execution time.

## 4. Missing deduplication
**Wrong:**
```
# Webhook fires 3 times for same alert → 3 identical cases created
```
**Right:** Add a check step:
```yaml
# Before create_case:
type: core.script.run_python
inputs:
  script: |
    def main(alert_id, existing_cases):
        return {"exists": any(c.get("alert_id") == alert_id for c in existing_cases)}
```
**Why:** Webhooks can fire multiple times for the same event. Deduplicate to avoid noise.

## 5. Monolithic workflows
**Wrong:** 30 actions in a single workflow doing triage + enrichment + containment + notification.
**Right:** Break into sub-workflows:
```
Main workflow → core.workflow.execute (enrichment sub-workflow)
             → core.workflow.execute (containment sub-workflow)
             → core.workflow.execute (notification sub-workflow)
```
**Why:** Modular workflows are easier to debug, test, and reuse.

## 6. Not setting trigger position
**Wrong:** Only position actions, forget the trigger.
**Right:**
```
tracecat_update_trigger_position({ workflow_id: "wf_xxx", x: 500, y: 0 })
```
**Why:** The trigger also needs to be positioned for a clean graph layout.

## 7. Branching without join
**Wrong:**
```
Score → High path → ...
     → Low path  → ...
# No merge point — two dead ends
```
**Right:**
```
Score → High path → Log result (join_strategy: any)
     → Low path  → Log result
```
**Why:** Branches should converge at a merge point for clean workflow completion.

## 8. Hardcoded thresholds
**Wrong:**
```yaml
run_if: ${{ ACTIONS.score.result.vt_malicious > 5 }}
```
**Better:**
```yaml
# Use trigger input for configurable threshold
run_if: ${{ ACTIONS.score.result.vt_malicious > TRIGGER.data.threshold }}
```
**Why:** Configurable thresholds make workflows reusable across different alert sources.

## 9. "Expected 1 logical entrypoint, got 2" validation error
**Wrong:** Multiple actions with no upstream connection (both look like starting points):
```
Trigger → Action A → Action C
          Action B → Action D   ← B has no upstream = second entrypoint
```
**Right:** Every non-trigger action must have at least one incoming edge:
```
Trigger → Action A → Action C
        → Action B → Action D
```
**Why:** Tracecat expects exactly one logical entrypoint (the trigger). If multiple actions have no incoming edge, the validator raises this error. Fix by connecting all actions to the trigger or to another action.

## 10. Not using child workflows for complex loops
**Wrong:** One massive workflow with `for_each` doing 10+ actions per iteration.
**Right:** Use `core.workflow.execute` to delegate loop body to a child workflow:
```yaml
# Parent workflow
- ref: process_each_alert
  action: core.workflow.execute
  args:
    workflow_id: wf_child_workflow_id
    payload:
      alert: ${{ var.alert }}
  control_flow:
    for_each: ${{ ACTIONS.get_alerts.result.items }}
```
**Why:** Child workflows keep logic modular, allow independent testing, and avoid hitting graph complexity limits.

## 11. Parallel nodes too close together
**Wrong:**
```
List Alerts: (500, 300)
Get Details: (340, 460)   Search Endpoint: (660, 460)   ← only 320px apart
Create Case: (500, 620)
```
**Right:**
```
List Alerts: (500, 300)
Get Details: (200, 500)   Search Endpoint: (800, 500)   ← 600px apart
Create Case: (500, 700)
```
**Why:** Nodes are ~250px wide. At 320px apart, they nearly touch or overlap. Use **350px minimum** between siblings — ideally ±300 from center (600px total). Also use consistent 200px vertical gaps, not 160px.

## 12. SIEM severity mapping without standardization
**Wrong:** Passing raw SIEM severity strings directly to case priority:
```yaml
priority: ${{ TRIGGER.data.severity }}
# "CRITICAL" ≠ "critical" — Tracecat rejects uppercase
```
**Right:** Map SIEM severity to Tracecat priority with `FN.conditional`:
```yaml
priority: ${{ FN.conditional(TRIGGER.data.severity == "critical" || TRIGGER.data.severity == "CRITICAL", "critical", FN.conditional(TRIGGER.data.severity == "high" || TRIGGER.data.severity == "HIGH", "high", "medium")) }}
```
**Why:** Different SIEMs use different severity formats. Always normalize to Tracecat's expected values: `low`, `medium`, `high`, `critical`.
