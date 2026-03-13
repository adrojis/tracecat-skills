---
name: tracecat-validation-debug
description: Activate when users debug failed workflow executions, troubleshoot errors, or validate workflows before deployment
---

# Tracecat Validation & Debug Expert

You are an expert at debugging Tracecat workflow executions, interpreting errors, and guiding users through validation and troubleshooting cycles.

## Execution Status Reference

### Workflow-Level Status
| Status | Meaning | Action |
|---|---|---|
| `RUNNING` | In progress | Wait or check individual action events |
| `COMPLETED` | All actions succeeded (or handled via error paths) | Review results |
| `FAILED` | Unhandled error occurred | Inspect `failure` field on events |
| `TIMED_OUT` | Exceeded timeout | Check `workflow_timeout` or slow actions |
| `CANCELED` | Manually stopped via API/UI | Intentional — no fix needed |
| `TERMINATED` | Forcibly killed | Check system logs |

### Action-Level Status
| Status | Meaning |
|---|---|
| `SCHEDULED` | Queued, waiting for dependencies |
| `STARTED` | Currently executing |
| `COMPLETED` | Finished successfully |
| `FAILED` | Error occurred — check `action_error` |
| `TIMED_OUT` | Action exceeded its timeout |
| `CANCELED` / `TERMINATED` | Stopped externally |

## Error Types

### Expression Errors (`TracecatExpressionError`)
- **Cause:** Invalid `${{ }}` syntax, bad JSONPath, wrong action ref
- **Symptom:** `Unexpected end-of-input. Expected one of...`
- **Fix:** Verify expression syntax matches `${{ ACTIONS.<slug>.result.<path> }}`. Check action slugs are the slugified version of the action title.

### Registry Validation Errors (`RegistryValidationError`)
- **Cause:** Invalid action type reference, missing required fields
- **Symptom:** Commit/deploy fails with `ok: false` in validation response
- **Fix:** Verify action types exist in registry. Check `errors` array in commit response.

### HTTP Action Errors
- **Cause:** External API returned error status code
- **Symptom:** Action fails with 4xx/5xx
- **Fix:** For expected failures (4xx), use **error paths** instead of failing. Access error data via `${{ ACTIONS.<ref>.error }}`.

### Secret Access Errors
- **Cause:** Missing secret, wrong key name, expired credential
- **Symptom:** Workflow fails on `SECRETS.<name>.<key>` resolution
- **Fix:** Verify secret exists in workspace with correct key name via `tracecat_search_secrets`.

### Child Workflow Errors
- **Cause:** Error in sub-workflow propagated up
- **Symptom:** Nested error chain (ChildWorkflowFailure > ActivityFailure > original)
- **Fix:** Inspect innermost `cause` in `EventFailure` chain.

## Debug Workflow (Step-by-Step)

### 1. Identify the failure
```
tracecat_list_executions → find the failed execution
tracecat_get_execution → get full event history
```

### 2. Read the error
In the execution events, look for:
- `status: FAILED` on the event
- `failure.message` — human-readable error description
- `failure.cause` — original exception and stack trace
- In compact view: `action_error.message`

### 3. Locate the root cause
- **Which action failed?** Check `action_ref` and `action_name` on the failed event
- **What was the input?** Check `action_input` on the event
- **What did upstream actions return?** Check `action_result` on preceding events

### 4. Common fix patterns
| Error Pattern | Likely Cause | Fix |
|---|---|---|
| `TracecatExpressionError` | Typo in `${{ }}` expression | Fix syntax, check action slugs |
| `KeyError` on nested access | Upstream action returned different structure | Inspect actual `action_result` of upstream |
| HTTP 401/403 | Bad or expired credentials | Rotate secret via `tracecat_create_secret` |
| HTTP 404 | Wrong endpoint or missing resource | Check URL and payload |
| HTTP 422 | Invalid request body | Check field names and types |
| `TIMED_OUT` | Slow external API or large payload | Increase `timeout_seconds` or optimize |
| Commit fails with no detail | Workflow has no actions or no trigger | Add at least one action and trigger |

## Deploy-Time Validation

When deploying via `tracecat_deploy_workflow`, the system validates:

1. **Action registry** — Each action type must exist in the registry
2. **Expression syntax** — All `${{ }}` expressions in `run_if`, `for_each`, `retry_until`, and inputs
3. **Graph connectivity** — Checks for unreachable nodes at join points

The commit response contains:
- `status`: `"success"` or `"failure"`
- `message`: What happened
- `errors[]`: Per-action validation results with `ok`, `message`, `detail`, `action_ref`

## Retry Policies

| Field | Default | Description |
|---|---|---|
| `retry_policy.max_attempts` | 1 | Total attempts. `0` = unlimited. `1` = no retry. |
| `retry_policy.timeout` | 300s | Max duration for all retries combined |
| `retry_until` | null | Expression evaluated against result. Retries until `true`. |
| `start_delay` | 0 | Pre-execution delay in seconds |

Example retry until acknowledgment:
```yaml
control_flow:
  retry_policy:
    max_attempts: 5
    timeout: 600
  retry_until: ${{ ACTIONS.send_reminder.result.ack == True }}
```

## Error Paths (Branching on Failure)

Since PR #471, actions can define **success and error handles**:
- On success → next action in the normal flow
- On error → branch to an error-handling action

Access error data from a failed upstream action:
```
${{ ACTIONS.<failed_action_ref>.error }}
```

## Join Strategies

| Strategy | Behavior |
|---|---|
| `"all"` (default) | Downstream runs only if ALL upstream actions succeed |
| `"any"` | Downstream runs if AT LEAST ONE upstream succeeds |

Use `"any"` for error-tolerant branches where partial results are acceptable.

## Tips

- Always check `tracecat_health_check` first to rule out connectivity issues
- Use compact execution view for quick triage, full view for deep investigation
- Expression errors are the #1 cause of failures — double-check action slugs
- For flaky integrations, add retry policies rather than error paths
- For expected failures (API 404 = "not found"), use error paths to handle gracefully
- Secrets are purged from memory after execution — they don't persist in logs

## Related Skills
- **tracecat-mcp-tools-expert** — MCP tool reference (use `tracecat_validate_workflow` before deploying)
- **tracecat-workflow-patterns** — Correct workflow design patterns
- **tracecat-yaml-syntax** — Fix YAML/expression syntax errors
- **tracecat-integration-expert** — Debug integration-specific errors
- **tracecat-code-python** — Debug Python script failures

## Reference Files
- [Common Mistakes](./COMMON_MISTAKES.md)
- [Examples](./EXAMPLES.md)
- [README](./README.md)
