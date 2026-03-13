# Tracecat Error Catalog

## Expression Errors

### Invalid action reference
```
TracecatExpressionError: Unexpected end-of-input. Expected one of: identifier
Expression: ${{ ACTIONS.nonexistent_step.result }}
```
**Cause:** The action slug `nonexistent_step` doesn't match any action in the workflow.
**Fix:** Action slugs are the slugified version of the action title. Check `tracecat_list_actions` for correct refs.

### Bad JSONPath access
```
KeyError: 'attributes'
Expression: ${{ ACTIONS.get_data.result.attributes.name }}
```
**Cause:** The upstream action's result doesn't contain the `attributes` key.
**Fix:** Inspect the actual `action_result` of `get_data` in the execution events. The structure may differ from what you expect.

### Missing closing braces
```
TracecatExpressionError: Unexpected token
Expression: ${{ ACTIONS.step1.result.data }
```
**Fix:** Ensure expressions use `${{ ... }}` with double closing braces.

## Commit/Deploy Errors

### Empty workflow
```
status: failure
message: Error committing Workflow
errors: []
```
**Cause:** Workflow has no actions or no trigger configured.
**Fix:** Add at least one action with an entrypoint and a trigger.

### Invalid action type
```
errors: [{ ok: false, message: "Action type not found", action_ref: "step_1" }]
```
**Cause:** The action type (e.g., `tools.nonexistent.action`) doesn't exist in the registry.
**Fix:** Check available action types. For custom integrations, verify the registry is loaded.

### Invalid field constraints
```
HTTPValidationError 422
loc: ["body", "inputs", "url"]
msg: "value is not a valid URL"
```
**Cause:** API-level Pydantic validation failure.
**Fix:** Check the field value matches the expected type and format.

## Runtime Errors

### HTTP action failures
```
action_error: { message: "HTTP 401 Unauthorized" }
```
**Fix:** Rotate or verify the secret used. Check `${{ SECRETS.<name>.<key> }}` references.

```
action_error: { message: "HTTP 429 Too Many Requests" }
```
**Fix:** Add a `start_delay` between actions or reduce `for_each` batch size.

### Timeout errors
```
status: TIMED_OUT
```
**At action level:** Increase `timeout_seconds` (default 30s for Python scripts, 300s for retry policy).
**At workflow level:** Check for blocking actions or infinite retry loops.

### Python script errors
```
action_error: { message: "Script execution failed" }
```
**Note:** Detailed Python traces are suppressed for security (GitHub issue #1347). Debug by:
1. Simplifying the script to isolate the error
2. Adding `print()` statements (output may appear in logs)
3. Testing the script locally with the same inputs
4. Checking `dependencies` are listed and `allow_network: true` is set if needed

## Secret Errors

### Secret not found
```
Error resolving secret: SECRETS.my_api.key
```
**Fix:** Verify the secret exists: `tracecat_search_secrets` with name `my_api`. Create it if missing.

### Wrong key name
```
KeyError: 'api_key' not in secret 'virustotal'
```
**Fix:** Secret keys are case-sensitive. Check the exact key names when the secret was created.

## Graph/Flow Errors

### Unreachable action
Commit validation may warn about actions that cannot be reached due to graph connectivity.
**Fix:** Ensure every action has a valid `depends_on` chain from the entrypoint.

### Join deadlock
If using `join_strategy: "all"` with parallel branches where one branch can fail:
**Fix:** Switch to `join_strategy: "any"` or add error paths to handle branch failures.
