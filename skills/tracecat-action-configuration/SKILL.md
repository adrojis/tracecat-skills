---
name: tracecat-action-configuration
description: Configure Tracecat workflow actions correctly. Use when setting up HTTP requests, transforms, AI actions, Python scripts, loops, conditions, retry policies, or any action type in a Tracecat workflow.
---

# Tracecat Action Configuration

## Core Action Types

### HTTP Request (`core.http_request`)

| Parameter | Type | Required | Default |
|-----------|------|----------|--------|
| `url` | str | Yes | -- |
| `method` | GET/POST/PUT/PATCH/DELETE | Yes | -- |
| `headers` | dict | No | -- |
| `params` | dict | No | -- |
| `payload` | dict/list | No | -- |
| `form_data` | dict | No | -- |
| `auth` | dict | No | -- |
| `timeout` | float | No | 10.0 |
| `follow_redirects` | bool | No | false |
| `verify_ssl` | bool | No | true |

**IMPORTANT**: Use `payload` NOT `body` for request body.

Returns: `{status_code, headers, data}`

### HTTP Polling (`core.http_poll`)

Same as `core.http_request` plus:

| Parameter | Type | Default |
|-----------|------|--------|
| `poll_retry_codes` | list[int] | -- |
| `poll_interval` | float | -- |
| `poll_max_attempts` | int | 10 |
| `poll_condition` | str (lambda) | -- |

```yaml
poll_retry_codes: [404]
poll_interval: 5
poll_condition: "lambda x: x['data'].get('status') == 'completed'"
```

### HTTP Pagination (`core.http_paginate`)

| Parameter | Type | Required |
|-----------|------|----------|
| `stop_condition` | str (lambda) | Yes |
| `next_request` | str (lambda) | Yes |
| `items_jsonpath` | str | No |
| `limit` | int | No (default 1000) |

### Reshape (`core.transform.reshape`)

Single `value` parameter. Use for:
- Renaming fields
- Extracting nested data
- Combining data from multiple actions
- Debugging (reshape = print)

```yaml
value:
  ip: ${{ TRIGGER.src_ip }}
  is_public: ${{ FN.ipv4_is_public(TRIGGER.src_ip) }}
  timestamp: ${{ FN.to_isoformat(FN.now()) }}
```

### Transform Actions

| Action | Use case | Key params |
|--------|----------|------------|
| `core.transform.filter` | Filter list items | `items`, `python_lambda` |
| `core.transform.apply` | Transform single value | `value`, `python_lambda` |
| `core.transform.map` | Transform each item | `items`, `python_lambda` |
| `core.transform.deduplicate` | Remove duplicates | `items`, `keys` |
| `core.transform.is_in` | Check membership | `items`, `collection` |
| `core.transform.not_in` | Check non-membership | `items`, `collection` |
| `core.transform.scatter` | Split list to items | `collection` |
| `core.transform.gather` | Merge items to list | `items` |

**Lambda syntax** (always use `>-` block modifier):
```yaml
python_lambda: >-
  lambda x: x['severity'] == 'high'
```

### Python Script (`core.script.run_python`)

| Parameter | Type | Default |
|-----------|------|--------|
| `script` | str (Python code) | required |
| `inputs` | dict | -- |
| `dependencies` | list[str] | -- |
| `timeout_seconds` | int | 30 |
| `allow_network` | bool | false |

Rules:
- Script MUST contain at least one function
- Multiple functions -> one must be named `main`
- Return must be JSON-serializable
- Input keys must match function parameter names
- Network disabled by default

```yaml
script: |
  def main(alert_data):
      if not alert_data:
          return {"success": False, "error": "No data"}
      score = len(alert_data.get("indicators", []))
      return {"success": True, "risk_score": score}
inputs:
  alert_data: ${{ ACTIONS.get_alert.result }}
dependencies:
  - requests
allow_network: true
```

**Alternative: Inject data via template expressions** (no `inputs` needed):

```yaml
script: |
  import json

  def process():
      data = json.loads("""${{ FN.serialize_json(ACTIONS.previous_action.result) }}""")
      # Process data...
      return {"processed": True, "count": len(data)}
```

**CRITICAL**: Top-level `return` is NOT allowed. Code MUST be inside a function.
Wrong: `return {"result": 42}` at module level.
Right: `def main(): return {"result": 42}`

### AI Actions

| Action | Use case |
|--------|----------|
| `ai.action` | Single LLM call with prompt |
| `ai.agent` | LLM with tool calling |

```yaml
# Simple AI analysis
- ref: ai_triage
  action: ai.action
  args:
    user_prompt: |
      Analyze this alert and return JSON:
      {"risk_score": 1-10, "summary": "...", "recommendations": [...]}
      Alert: ${{ FN.serialize_json(ACTIONS.get_alert.result) }}
    model_name: gpt-4o-mini
    model_provider: openai
```

**Tip**: Use gpt-4o-mini for high-volume triage (cheaper, fast enough).
**Tip**: Convert JSON to YAML before sending to AI (20-30% fewer tokens).

### Child Workflow (`core.workflow.execute`)

| Parameter | Type | Default |
|-----------|------|--------|
| `workflow_alias` | str | -- |
| `workflow_id` | str | -- |
| `trigger_inputs` | dict | -- |
| `wait_strategy` | wait/detach | detach |
| `loop_strategy` | parallel/batch/sequential | -- |
| `batch_size` | int | -- |
| `fail_strategy` | isolated/all | -- |

```yaml
- ref: run_enrichment
  action: core.workflow.execute
  for_each: ${{ for var.ip in ACTIONS.extract_ips.result }}
  args:
    workflow_alias: enrich_ip
    trigger_inputs:
      ip: ${{ var.ip }}
    wait_strategy: wait
    loop_strategy: parallel
    batch_size: 5
    fail_strategy: isolated
```

### Require (`core.require`)

Gate that validates conditions before proceeding:

```yaml
- ref: validate
  action: core.require
  args:
    conditions:
      - ${{ FN.length(ACTIONS.results.result) > 0 }}
      - ${{ ACTIONS.auth.result.status_code == 200 }}
    raise_error: true
    require_all: true
```

## Control Flow

### If/Else Branching — Decision hierarchy (prefer top options)

**1. Filters + run_if (BEST — visual, no code)**

Use `core.transform.filter` to split data into branches, then `run_if` on downstream actions:

```yaml
# Filter splits the list by condition
- ref: filter_critical
  action: core.transform.filter
  args:
    items: ${{ ACTIONS.extract_stats.result }}
    python_lambda: >-
      lambda x: x['score'] >= 16

# Case only runs if filter found results
- ref: case_critical
  action: core.cases.create_case
  depends_on: [filter_critical]
  run_if: ${{ FN.length(ACTIONS.filter_critical.result) > 0 }}
  for_each: ${{ for var.item in ACTIONS.filter_critical.result }}
  args:
    summary: "CRITICAL - ${{ var.item.ip }}"
    priority: critical
    status: new
```

This creates a visual tree in the UI where only active branches execute.

**2. Reshape with ternary (OK — inline, single action)**

```yaml
value:
  priority: "${{ 'critical' if score >= 16 else ('high' if score >= 6 else 'low') }}"
```

Use when you need a computed field but don't need visual branching.

**3. Python script (LAST RESORT — when logic is too complex)**

Use `core.script.run_python` only when filter/reshape can't express the logic.

### CRITICAL: run_if does NOT support loop variables

`run_if` with `var.*` inside a `for_each` **FAILS**. This does NOT work:
```yaml
# WRONG — will error
run_if: ${{ var.item.score >= 16 }}
for_each: ${{ for var.item in ACTIONS.data.result }}
```

Instead, use `FN.length()` on an upstream filter result:
```yaml
# CORRECT
run_if: ${{ FN.length(ACTIONS.filter_critical.result) > 0 }}
for_each: ${{ for var.item in ACTIONS.filter_critical.result }}
```

### for_each on empty list = COMPLETED (green)

A `for_each` over an empty list `[]` completes with 0 iterations and shows green in the UI. To prevent this, add a `run_if` that checks the list length first.

### If Conditions

```yaml
run_if: ${{ ACTIONS.check.result.status == 200 }}
run_if: ${{ ACTIONS.check.result.success == true && ACTIONS.check.result.count > 0 }}
run_if: ${{ ACTIONS.severity.result in ["high", "critical"] }}
run_if: ${{ ACTIONS.field.result != None }}
run_if: ${{ FN.length(ACTIONS.filter.result) > 0 }}
```

### Loops

```yaml
for_each: ${{ for var.alert in ACTIONS.list_alerts.result[*] }}
# Access: ${{ var.alert.id }}, ${{ var.alert.severity }}
```

Results from looped actions become a **list** (one entry per iteration).

### Retry Policy

```yaml
retry_policy:
  max_attempts: 3
  timeout: 60
  retry_until: ${{ ACTIONS.check_status.result.complete == true }}
```

### Wait Until

Uses `dateparser`: `"tomorrow at 2pm"`, `"in 2 hours"`, `"2025-05-01 at 10am"`

## Expression Reference

### Contexts

| Context | Syntax | Example |
|---------|--------|--------|
| Action output | `ACTIONS.ref.result` | `${{ ACTIONS.get_data.result.items[0] }}` |
| Trigger data | `TRIGGER` | `${{ TRIGGER.alert_id }}` |
| Secrets | `SECRETS.name.key` | `${{ SECRETS.splunk.SPLUNK_API_KEY }}` |
| Functions | `FN.func()` | `${{ FN.now() }}` |
| Loop var | `var.name` | `${{ var.item.field }}` |
| Workspace vars | `VARS.ns.key` | `${{ VARS.splunk.base_url }}` |

### Operators

| Operator | Example |
|----------|--------|
| Comparison | `==`, `!=`, `>`, `>=`, `<`, `<=` |
| Logical | `&&`, `\|\|` |
| Membership | `in`, `not in` |
| Identity | `is`, `is not`, `== None` |
| Arithmetic | `+`, `-`, `*`, `/`, `%` |
| Ternary | `${{ "yes" if x else "no" }}` |
| Default | `${{ value \|\| "fallback" }}` |

### Typecasting

```yaml
${{ int("42") }}
${{ str(42) }}
${{ "42" -> int }}
${{ bool("true") }}
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `body:` in HTTP action | Use `payload:` |
| Nested `${{ ${{ }} }}` | Use `${{ FN.func1(FN.func2()) }}` |
| Lambda with double quotes inside YAML string | Use `>-` block modifier |
| Missing `[*]` for list JSONPath | `result.data[*].name` not `result.data.name` |
| `for_each` with single item | String gets iterated char-by-char; use `core.http_request` directly |
| Dots in field names | Quote them: `result."kibana.alert.rule.name"` |
| AI returns free text | Use schema-constrained JSON output |