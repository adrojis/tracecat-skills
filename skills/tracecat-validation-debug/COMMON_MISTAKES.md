# Common Mistakes — Validation & Debug

## 1. Not checking execution details
**Wrong:** "The workflow failed" → immediately trying to fix code.
**Right:**
```
tracecat_get_execution_compact({ execution_id: "exec_xxx" })
```
Then find the specific action that failed and read its error message.
**Why:** Always identify the exact failing action and error before attempting fixes.

## 2. Checking execution too early
**Wrong:**
```
tracecat_run_workflow({ workflow_id: "wf_xxx" })
tracecat_get_execution({ execution_id: "exec_xxx" })  # Immediately
# Status: "running"
```
**Right:** Wait a few seconds, or check repeatedly:
```
tracecat_run_workflow(...)
# Wait briefly
tracecat_get_execution_compact({ execution_id: "exec_xxx" })
```
**Why:** Executions are async. Checking immediately shows "running", not the final result.

## 3. Ignoring warnings from validate
**Mistake:**
```
tracecat_validate_workflow(...)
→ { valid: true, warnings: ["Action X has empty inputs", "Action Y is orphaned"] }
# "It says valid, ship it!"
```
**Right:** Warnings indicate potential issues. `valid: true` only means no hard errors. Fix warnings too.

## 4. Deploying without validating
**Wrong:**
```
tracecat_deploy_workflow({ workflow_id: "wf_xxx" })
# Hope it works!
```
**Right:**
```
tracecat_validate_workflow({ workflow_id: "wf_xxx" })
# Fix any errors
tracecat_deploy_workflow({ workflow_id: "wf_xxx" })
```
**Why:** Validation catches expression errors, orphan nodes, and missing connections before deployment.

## 5. Wrong error path interpretation
**Wrong:** Assuming `ACTIONS.x.error` means the action returned an error value.
**Right:** `ACTIONS.x.error` is only populated when the action FAILED (threw an exception). If the action succeeded with error-like data, use `ACTIONS.x.result`.

## 6. Retry policy without timeout
**Wrong:**
```yaml
retry_policy:
  max_attempts: 10
```
**Right:**
```yaml
retry_policy:
  max_attempts: 3
  timeout: 60
```
**Why:** Without timeout, retries can run indefinitely. Always set both max_attempts and timeout.

## 7. Not using autofix before manual debugging
**Slower approach:** Manually inspect graph, find orphans, add edges one by one.
**Faster approach:**
```
tracecat_autofix_workflow({ workflow_id: "wf_xxx", dry_run: true })
# Review planned fixes
tracecat_autofix_workflow({ workflow_id: "wf_xxx" })
```
**Why:** Autofix handles the most common issues (orphans, disconnected trigger, unclosed expressions) automatically.

## 8. Fixing symptoms instead of root cause
**Wrong:** Action fails with "unknown slug" → rename the action.
**Right:** Check that the referenced action slug actually exists and matches the `ref` field:
```
tracecat_list_actions({ workflow_id: "wf_xxx" })
# Verify slugs match expression references
```
**Why:** "Unknown slug" means the expression references a non-existent action ref, not a naming issue.

## 9. "Expected 1 logical entrypoint, got 2" error
**Symptom:** Validation fails with `Expected 1 logical entrypoint, got 2` (or more).
**Cause:** Multiple actions have no incoming edge — they all look like starting points.
**Fix:**
1. Run `tracecat_get_graph({ workflow_id: "wf_xxx" })` to inspect edges
2. Identify actions with no incoming connection
3. Connect them to the trigger or to an upstream action via `tracecat_add_edges`
4. Or run `tracecat_autofix_workflow` which often fixes this automatically

## 10. VirusTotal base64 padding error
**Symptom:** VT URL lookup returns 404 or "invalid resource" error.
**Cause:** `FN.to_base64url()` produces trailing `==` padding that VirusTotal rejects.
**Fix:** Always wrap with `FN.strip()`:
```yaml
# Wrong
url: https://www.virustotal.com/api/v3/urls/${{ FN.to_base64url(TRIGGER.data.url) }}

# Right
url: https://www.virustotal.com/api/v3/urls/${{ FN.strip(FN.to_base64url(TRIGGER.data.url)) }}
```
