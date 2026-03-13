---
name: tracecat-integration-builder
description: Build a complete Tracecat integration for ANY technology from just a URL. Use when the user provides a documentation URL, API reference, or product page and wants to integrate that service into Tracecat with actions, secrets, and workflows.
---

# Tracecat Integration Builder

You build complete Tracecat integrations from scratch. The user gives you a URL (docs, API reference, product page — anything), and you deliver a working integration: secrets, actions, edges, positioning, ready to use.

## Critical Rules

1. **The user gives a URL, you do everything else** — research the API, figure out auth, build the actions, deploy
2. **Auth ALWAYS goes through Tracecat secrets** — `${{ SECRETS.xxx.KEY }}`, never hardcoded
3. **Use native Tracecat action types first** — check if `tools.<service>.*` already exists before building custom actions
4. **Always connect edges and position nodes** — every integration must be a deployable workflow, not orphan actions
5. **Ask for credentials, never guess them** — prompt the user for API keys/tokens, then create the secret via MCP

## Process

### Step 1: Research the service (YOU do this, not the user)

When the user gives you a URL:

1. **Fetch the URL** with WebFetch to understand what the service is
2. **Search for API documentation** — look for:
   - REST API reference / endpoints list
   - Authentication method (API key, OAuth2, Basic, JWT, custom header)
   - Rate limits and pagination
   - OpenAPI/Swagger spec if available
3. **Search for an OpenAPI spec** — check:
   - `{base_url}/openapi.json`, `{base_url}/swagger.json`, `{base_url}/api-docs`
   - GitHub repos of the service (often have specs in `/docs` or `/api`)
   - Community specs on https://apis.guru or GitHub
4. **Determine the SOAR-relevant endpoints** — for security tools, focus on:
   - Alerts / detections / events (GET)
   - Assets / devices / users (GET)
   - Containment / isolation / blocking (POST)
   - IOC submission / lookup (GET/POST)
   - Ticket / case creation (POST)

### Step 2: Determine the integration strategy

Based on your research, pick the right approach:

| Situation | Strategy |
|-----------|----------|
| OpenAPI 3.0 spec available | Use Tracecat OpenAPI Converter (fastest) |
| OpenAPI 2.0 (Swagger) spec | Convert to 3.0, then use converter |
| REST API with good docs, no spec | Build actions manually with `core.http_request` |
| Webhook/push-based service | Configure Tracecat webhook trigger |
| Non-HTTP protocol (syslog, gRPC) | Use `core.script.run_python` with appropriate libs |
| Service already has native Tracecat integration | Just create the secret and use existing `tools.*` actions |

### Step 3: Build the integration

#### Path A — OpenAPI Converter (if spec exists)

1. Download/fetch the OpenAPI spec
2. Create the conversion config YAML:
```yaml
endpoints:
  include:
    like:
      - "/relevant/endpoints/*"
  exclude:
    like:
      - "/auth/*"
      - "/admin/*"

definition_overrides:
  namespace: "tools.<service>"
  display_group: "<Service Name>"

auth:
  secrets:
    - name: "<service>"
      keys: ["API_KEY"]  # or CLIENT_ID, CLIENT_SECRET, etc.
  injection:
    args:
      headers:
        Authorization: "Bearer ${{ SECRETS.<service>.API_KEY }}"
```
3. Run: `uv run scripts/openapi_to_template.py --input <spec> --output-dir <dir> --config <config>`
4. Import generated templates into Tracecat

#### Path B — Manual build (most common for security tools)

Build the full workflow directly via MCP tools:

1. **Create the workflow**
```
tracecat_create_workflow:
  title: "<Service> Integration"
  description: "Automated <service> integration"
```

2. **Create the secret** — ask the user for credentials first
```
tracecat_create_secret:
  name: "<service>"
  keys:
    - key: "API_KEY"
      value: "<user-provided-value>"
```

3. **Create actions** for each endpoint needed
```
tracecat_create_action:
  workflow_id: "wf_xxx"
  type: "core.http_request"
  title: "Get <Service> Alerts"
```

4. **Configure action inputs** (YAML string!)
```
tracecat_update_action:
  action_id: "act_xxx"
  workflow_id: "wf_xxx"
  inputs: |
    url: https://api.service.com/v1/alerts
    method: GET
    headers:
      Authorization: Bearer ${{ SECRETS.service.API_KEY }}
      Accept: application/json
    params:
      limit: 100
```

5. **Connect edges** — link trigger → actions → downstream
```
tracecat_add_edges:
  workflow_id: "wf_xxx"
  edges:
    - source_id: "<trigger_id>"
      target_id: "<first_action_id>"
      source_type: "trigger"
```

6. **Position nodes** for clean visual layout
```
tracecat_move_nodes:
  workflow_id: "wf_xxx"
  positions:
    - action_id: "<action_id>"
      x: 500
      y: 300
```

7. **Validate and deploy**
```
tracecat_validate_workflow: { workflow_id: "wf_xxx" }
tracecat_deploy_workflow: { workflow_id: "wf_xxx" }
```

#### Path C — Webhook trigger (push-based services)

For services that push events to Tracecat (Splunk alerts, Wazuh events, etc.):

1. Create workflow with webhook trigger
2. Enable the webhook: status "online"
3. Give the user the webhook URL to configure in the source service
4. Build downstream processing actions (enrichment, case creation, etc.)

### Step 4: Test the integration

1. **Validate** — `tracecat_validate_workflow`
2. **Dry run** — `tracecat_run_workflow` with test payload
3. **Check execution** — `tracecat_get_execution_compact` to verify each action succeeded
4. **Debug if needed** — check error messages, fix inputs, re-validate

## Auth Patterns Quick Reference

### API Key (header)
```yaml
headers:
  x-apikey: ${{ SECRETS.virustotal.API_KEY }}
```

### API Key (query param)
```yaml
params:
  api_key: ${{ SECRETS.service.API_KEY }}
```

### Bearer Token
```yaml
headers:
  Authorization: Bearer ${{ SECRETS.service.TOKEN }}
```

### Basic Auth
```yaml
headers:
  Authorization: Basic ${{ FN.to_base64(SECRETS.service.USER + ':' + SECRETS.service.PASSWORD) }}
```

### OAuth2 (requires token refresh workflow)
```yaml
# Step 1: Token request action
url: https://auth.service.com/oauth2/token
method: POST
headers:
  Content-Type: application/x-www-form-urlencoded
payload:
  grant_type: client_credentials
  client_id: ${{ SECRETS.service.CLIENT_ID }}
  client_secret: ${{ SECRETS.service.CLIENT_SECRET }}

# Step 2: Use token in subsequent actions
headers:
  Authorization: Bearer ${{ ACTIONS.get_token.result.access_token }}
```

### Custom/JWT (Wazuh-style)
```yaml
# Step 1: Auth action to get JWT
url: ${{ SECRETS.wazuh.BASE_URL }}/security/user/authenticate
method: POST
headers:
  Authorization: Basic ${{ FN.to_base64(SECRETS.wazuh.API_USER + ':' + SECRETS.wazuh.API_PASSWORD) }}

# Step 2: Use JWT in subsequent actions
headers:
  Authorization: Bearer ${{ ACTIONS.auth.result.data.token }}
```

## Layout Rules (rappel)

| Element | Position |
|---------|----------|
| Trigger | x=500, y=0 |
| First action | x=500, y=300 |
| Vertical spacing | 160px between rows |
| Horizontal spacing | 320px between parallel nodes |
| Spine principal | x=500 |

## SOAR Use Case Templates

### Enrichment Pipeline
```
Trigger → Fetch IOC → Check TI Service → Score → Create Case (if malicious)
```

### Alert Triage
```
Webhook (alert) → Extract IOCs → Parallel Enrichment → Aggregate → Decision → Case/Notification
```

### Containment
```
Trigger → Verify Threat → Isolate Host → Block IOC → Create Case → Notify SOC
```

### Scheduled Check
```
Schedule (every 5min) → Query SIEM → Filter New Alerts → Enrich → Create Cases
```

## Related Skills
- **tracecat-action-configuration** — Action types, inputs format, control flow
- **tracecat-secrets-integrations** — Secret creation patterns per service
- **tracecat-workflow-patterns** — Workflow architecture patterns
- **tracecat-integration-expert** — Existing native integration reference
- **tracecat-mcp-tools-expert** — MCP tool usage for deployment
- **tracecat-validation-debug** — Debug integration issues

## Reference Files
- [Common Mistakes](./COMMON_MISTAKES.md)
- [Examples](./EXAMPLES.md)
- [README](./README.md)
