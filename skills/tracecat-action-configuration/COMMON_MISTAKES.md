# Common Mistakes — Action Configuration

## 1. Wrong action type separator
**Wrong:**
```yaml
type: core.http.request
```
**Right:**
```yaml
type: core.http_request
```
**Why:** Tracecat uses underscores for core action types. Dots are only for integration actions (`tools.virustotal.analyze_hash`).

## 2. Python script field name
**Wrong:**
```yaml
code: |
  def main():
      return {"result": True}
```
**Right:**
```yaml
script: |
  def main():
      return {"result": True}
```
**Why:** The field is `script`, not `code`. Using `code` silently ignores the script.

## 3. Inputs as JSON object via MCP
**Wrong:**
```
tracecat_update_action({
  inputs: { url: "https://...", method: "GET" }
})
```
**Right:**
```
tracecat_update_action({
  inputs: "url: https://...\nmethod: GET"
})
```
**Why:** The MCP `inputs` parameter must be a YAML string. JSON objects cause validation errors.

## 4. Boolean equality in run_if
**Wrong:**
```yaml
run_if: ${{ ACTIONS.check.result == true }}
run_if: ${{ ACTIONS.check.result == false }}
```
**Right:**
```yaml
run_if: ${{ ACTIONS.check.result }}
run_if: ${{ not ACTIONS.check.result }}
```
**Why:** Tracecat expressions use truthy/falsy evaluation. `== true` and `== false` can produce unexpected results.

## 5. Python logical operators in expressions
**Wrong:**
```yaml
run_if: ${{ ACTIONS.a.result and ACTIONS.b.result }}
run_if: ${{ ACTIONS.a.result or ACTIONS.b.result }}
```
**Right:**
```yaml
run_if: ${{ ACTIONS.a.result && ACTIONS.b.result }}
run_if: ${{ ACTIONS.a.result || ACTIONS.b.result }}
```
**Why:** Tracecat expressions use C-style operators (`&&`, `||`), not Python-style (`and`, `or`).

## 6. Using reshape for case operations
**Wrong:**
```yaml
type: core.transform.reshape
inputs:
  value:
    case_title: "New alert"
    status: "new"
# Then POST to /cases via http_request...
```
**Right:**
```yaml
type: core.cases.create_case
inputs:
  case_title: "New alert"
  status: new
  priority: medium
```
**Why:** Native case actions handle validation, workspace association, and field formatting automatically.

## 7. Missing allow_network for Python HTTP
**Wrong:**
```yaml
type: core.script.run_python
inputs:
  script: |
    import requests
    def main():
        return requests.get("https://api.example.com").json()
  dependencies:
    - requests
```
**Right:**
```yaml
type: core.script.run_python
inputs:
  script: |
    import requests
    def main():
        return requests.get("https://api.example.com").json()
  dependencies:
    - requests
  allow_network: true
```
**Why:** Python scripts run in a sandbox with network disabled by default.

## 8. Passing complex objects to Python without serialization
**Wrong:**
```yaml
inputs:
  data: ${{ ACTIONS.fetch.result }}
```
**Right:**
```yaml
inputs:
  data: ${{ FN.serialize_json(ACTIONS.fetch.result) }}
```
Then in Python: `import json; data = json.loads(data)`

**Why:** Complex nested objects may not serialize correctly as function arguments. `FN.serialize_json` ensures proper JSON string transfer.

## 9. Forgetting control_flow wrapper
**Wrong (via MCP):**
```
tracecat_update_action({
  run_if: "${{ ACTIONS.x.result }}"
})
```
**Right:**
```
tracecat_update_action({
  control_flow: {
    run_if: "${{ ACTIONS.x.result }}"
  }
})
```
**Why:** `run_if`, `for_each`, `retry_policy` etc. must be nested inside `control_flow`.

## 10. Wrong retry_policy structure
**Wrong:**
```yaml
control_flow:
  retry: 3
  timeout: 60
```
**Right:**
```yaml
control_flow:
  retry_policy:
    max_attempts: 3
    timeout: 60
```
**Why:** Retry configuration must be nested under `retry_policy` with specific field names.

## 11. Missing verify_ssl for on-premise APIs
**Wrong:**
```yaml
type: core.http_request
inputs:
  url: https://wazuh.internal:55000/agents
  method: GET
  headers:
    Authorization: Bearer ${{ SECRETS.wazuh.API_KEY }}
```
**Right:**
```yaml
type: core.http_request
inputs:
  url: ${{ SECRETS.wazuh.SERVER_URL }}/agents
  method: GET
  headers:
    Authorization: Bearer ${{ SECRETS.wazuh.API_KEY }}
  verify_ssl: false
```
**Why:** Self-hosted tools (Wazuh, MISP, TheHive) typically use self-signed certificates. Without `verify_ssl: false`, httpx raises an SSL verification error.

## 12. No timeout for slow external APIs
**Wrong:**
```yaml
type: core.http_request
inputs:
  url: https://slow-api.example.com/analyze
  method: POST
  payload:
    sample: ${{ TRIGGER.data.file_hash }}
```
**Right:**
```yaml
type: core.http_request
inputs:
  url: https://slow-api.example.com/analyze
  method: POST
  payload:
    sample: ${{ TRIGGER.data.file_hash }}
  timeout: 120
control_flow:
  retry_policy:
    max_attempts: 2
    timeout: 180
```
**Why:** Some APIs (sandbox analysis, SIEM queries) take 10+ seconds. Default httpx timeout may expire. Set explicit `timeout` in inputs and wrap with `retry_policy`.
