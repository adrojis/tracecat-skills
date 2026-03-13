---
name: tracecat-action-configuration
description: Activate when users configure Tracecat workflow actions — HTTP requests, transforms, AI actions, Python scripts, loops, conditions, retry policies, case operations, or any action type
---

# Tracecat Action Configuration Expert

You are an expert at configuring Tracecat workflow actions correctly. Use this reference for action types, required fields, inputs format, and control flow.

## Critical Rules

1. **ALWAYS use native/official action types first** — NEVER use `core.transform.reshape`, `core.http_request`, or `core.script.run_python` when a native action exists for the operation. This is the #1 rule.
   - Cases: use `core.cases.create_case`, `core.cases.update_case`, `core.cases.create_comment` — NOT reshape + http_request
   - Integrations: use `tools.virustotal.*`, `tools.crowdstrike.*`, `tools.okta.*`, `tools.splunk.*`, `tools.microsoft_defender.*`, `tools.sharepoint.*`, etc. — NOT core.http_request with manual auth headers
   - Only fall back to `core.http_request` when NO native integration exists for that API
   - Only use `core.transform.reshape` for data transformation/aggregation, NEVER for CRUD operations that have native actions
   - Only use `core.script.run_python` when the logic cannot be expressed with native actions or expressions
2. **Inputs are YAML strings** — when using the MCP `tracecat_update_action` tool, `inputs` must be a YAML string, NOT a JSON object
3. **Action type format** — use underscores: `core.http_request` (NOT `core.http.request`)
4. **Field name for Python** — the field is `script` (NOT `code`)
5. **Boolean expressions** — use `${{ ACTIONS.x.result }}` (truthy) and `${{ not ACTIONS.x.result }}` (falsy). NEVER use `== true` or `== false`
6. **Logical operators** — use `&&` and `||` (NOT `and`/`or`)

## Action Types Reference

### core.transform.reshape
Transform, aggregate, or restructure data. Most versatile action.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `value` | any | Yes | The value to output (expression or literal) |

```yaml
value:
  summary: ${{ ACTIONS.enrich.result.data.summary }}
  score: ${{ ACTIONS.score.result.risk_score }}
  is_malicious: ${{ ACTIONS.check.result.positives > 5 }}
```

**Use for:** reshaping API responses, building payloads, combining results from parallel actions, creating conditional values.

### core.http_request
Make HTTP requests to external APIs.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | Yes | Target URL |
| `method` | string | Yes | GET, POST, PUT, PATCH, DELETE |
| `headers` | object | No | HTTP headers |
| `payload` | object | No | Request body (JSON) |
| `params` | object | No | Query parameters |
| `timeout` | integer | No | Timeout in seconds (default: 30) |

```yaml
url: https://api.virustotal.com/api/v3/files/${{ ACTIONS.get_hash.result.sha256 }}
method: GET
headers:
  x-apikey: ${{ SECRETS.virustotal.API_KEY }}
  Accept: application/json
```

**Use for:** external API calls without a native integration. Always prefer native integrations when available.

### core.script.run_python
Execute Python code in a sandboxed environment.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `script` | string | Yes | Python code (must have a `main` function) |
| `inputs` | object | No | Key-value pairs passed as function arguments |
| `dependencies` | array | No | Pip packages to install |
| `timeout_seconds` | integer | No | Max execution time (default: 30) |
| `allow_network` | boolean | No | Enable outbound network (default: false) |

```yaml
script: |
  def main(items, threshold):
      return [i for i in items if i["score"] > threshold]
inputs:
  items: ${{ ACTIONS.fetch.result.data }}
  threshold: 50
```

**See tracecat-code-python skill for detailed patterns.**

### core.cases.create_case
Create a new case in case management.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `case_title` | string | Yes | Title of the case |
| `payload` | object | No | Case payload data |
| `status` | string | No | Initial status (new, in_progress, resolved, closed) |
| `priority` | string | No | Priority (low, medium, high, critical) |
| `malice` | string | No | Malice level (benign, malicious, unknown) |
| `action` | string | No | Recommended action |

```yaml
case_title: "Suspicious file detected: ${{ ACTIONS.analyze.result.filename }}"
payload:
  source_ip: ${{ TRIGGER.data.src_ip }}
  file_hash: ${{ TRIGGER.data.hash }}
  vt_score: ${{ ACTIONS.vt_check.result.positives }}
status: new
priority: ${{ ACTIONS.score.result.priority }}
malice: ${{ ACTIONS.score.result.malice }}
action: Investigate and contain
```

### core.cases.update_case
Update an existing case.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `case_id` | string | Yes | Case ID to update |
| `status` | string | No | New status |
| `priority` | string | No | New priority |
| `malice` | string | No | New malice level |
| `action` | string | No | New action |

### core.cases.create_comment
Add a comment to a case.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `case_id` | string | Yes | Case ID |
| `content` | string | Yes | Comment text |

### core.workflow.execute
Run a child workflow.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `workflow_id` | string | Yes | ID of the workflow to execute |
| `payload` | object | No | Input data for the child workflow |
| `timeout` | integer | No | Timeout in seconds |

### core.send_email
Send an email notification.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `to` | string/array | Yes | Recipient(s) |
| `subject` | string | Yes | Email subject |
| `body` | string | Yes | Email body (supports HTML) |

## Control Flow

### run_if — Conditional execution
```yaml
# Execute only if score is high
run_if: ${{ ACTIONS.score.result.risk > 70 }}

# Execute only for public IPs
run_if: ${{ FN.ipv4_is_public(TRIGGER.data.src_ip) }}

# Negate a boolean result
run_if: ${{ not ACTIONS.is_whitelisted.result }}

# Combine conditions
run_if: ${{ ACTIONS.is_malicious.result && ACTIONS.is_external.result }}
```

### for_each — Loop over items
```yaml
for_each: ${{ ACTIONS.get_iocs.result.indicators }}
```
When `for_each` is set, the action executes once per item. Access the current item via the action's normal input expressions.

### join_strategy — Parallel merge
- `all` (default) — Wait for all upstream actions
- `any` — Continue as soon as one upstream completes

### retry_policy — Auto-retry on failure
```yaml
control_flow:
  retry_policy:
    max_attempts: 3
    timeout: 60
```

### start_delay — Delay before execution
```yaml
control_flow:
  start_delay: 5  # seconds
```

## MCP Tool Usage for Actions

### Creating an action
```
tracecat_create_action:
  workflow_id: "wf_xxx"
  type: "core.http_request"
  title: "Check IP on VirusTotal"
```

### Updating action inputs (YAML string!)
```
tracecat_update_action:
  action_id: "act_xxx"
  workflow_id: "wf_xxx"
  inputs: "url: https://api.vt.com/v3/ip/${{ TRIGGER.data.ip }}\nmethod: GET\nheaders:\n  x-apikey: ${{ SECRETS.virustotal.API_KEY }}"
```

### Adding control flow
```
tracecat_update_action:
  action_id: "act_xxx"
  workflow_id: "wf_xxx"
  control_flow:
    run_if: "${{ ACTIONS.check.result.is_suspicious }}"
    retry_policy:
      max_attempts: 3
      timeout: 30
```

## Common Mistakes

| Mistake | Correct |
|---------|---------|
| `core.http.request` | `core.http_request` |
| `code: \|` in run_python | `script: \|` |
| `run_if: ${{ x == true }}` | `run_if: ${{ x }}` |
| `run_if: ${{ x and y }}` | `run_if: ${{ x && y }}` |
| inputs as JSON object in MCP | inputs as YAML string |
| Using reshape for cases | Use `core.cases.create_case` |
| Missing `allow_network` for HTTP in Python | Set `allow_network: true` |

## Built-in Functions (FN)

| Function | Description |
|----------|-------------|
| `FN.ipv4_is_public(ip)` | Check if IPv4 is public |
| `FN.str.upper(s)` | Uppercase string |
| `FN.str.lower(s)` | Lowercase string |
| `FN.serialize_json(obj)` | Serialize object to JSON string |
| `FN.deserialize_json(s)` | Parse JSON string to object |
| `FN.format.map_to_str(obj)` | Format map as readable string |

## Related Skills
- **tracecat-code-python** — Detailed Python scripting patterns
- **tracecat-workflow-patterns** — Workflow architecture using these actions
- **tracecat-yaml-syntax** — Expression syntax reference
- **tracecat-case-management** — Case lifecycle details
- **tracecat-secrets-integrations** — Secret configuration for action inputs
- **tracecat-mcp-tools-expert** — MCP tools for creating/updating actions
- **tracecat-validation-debug** — Debug action configuration errors

## Reference Files
- [Common Mistakes](./COMMON_MISTAKES.md)
- [Examples](./EXAMPLES.md)
- [README](./README.md)
