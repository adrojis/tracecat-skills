# Tracecat Integration Builder

Build Tracecat integrations to connect external APIs/tools. There are **two methods** — both use Tracecat's native secrets system and `core.http_request`. No custom OAuth, no separate MCP servers, no external auth middleware needed.

## When to Use

- User wants to add a new integration (e.g., "add HarfangLab", "integrate MISP")
- User has an OpenAPI spec and wants to generate action templates
- User asks about the OpenAPI converter or `openapi_to_template.py`
- User wants to connect a new API/tool to Tracecat

## Key Principle: Tracecat Handles Auth Natively

Tracecat integrations do NOT need:
- Custom OAuth flows or MCP servers per vendor
- External auth middleware or token refresh logic
- Separate credential stores

Instead, ALL integrations follow the same pattern:
1. **Create a Tracecat secret** with API keys/tokens (`tracecat_create_secret`)
2. **Reference the secret** in action templates via `${{ SECRETS.<name>.<KEY> }}`
3. **Use `core.http_request`** with the secret injected in headers/params

This works for API keys, Bearer tokens, Basic auth, custom headers — everything.

## Two Methods for Building Integrations

| | Path A: OpenAPI Converter | Path B: Manual Templates |
|---|---|---|
| **When** | Vendor publishes OpenAPI 3.0 spec | No spec available, or spec is incomplete |
| **How** | `openapi_to_template.py` auto-generates YAML | Write YAML action templates by hand |
| **Speed** | Fast (minutes) | Slower (research + write) |
| **Examples** | CrowdStrike, Jira, PagerDuty, Wazuh, VirusTotal | HarfangLab EDR, MISP (older), custom internal APIs |
| **Auth** | Injected via config YAML | Injected directly in template YAML |

Both methods produce the same output: YAML action templates that use `core.http_request` + Tracecat secrets.

## Path A: OpenAPI Converter

Tracecat's official tool for generating integrations from OpenAPI 3.0 specs.

### Command
```bash
uv run scripts/openapi_to_template.py --input <spec> --output-dir <dir> [--config <config.yaml>]
```

### Supported: OpenAPI 3.0 only
- OAS 2.0 (Swagger) and OAS 3.1 are NOT supported
- If the spec is Swagger 2.0, convert it first: https://converter.swagger.io/

## Config YAML — Full Reference

### Endpoints Filtering
```yaml
endpoints:
  include:
    like: ["/api/v1/alerts*", "/api/v1/threat*"]   # Glob patterns
    exact: ["/api/v1/status"]                        # Exact match
  exclude:
    like: ["/api/v1/admin/*", "/api/v1/internal/*"]
    exact: ["/api/v1/debug"]
```

**Strategy:** Start with `include.like` to select only the endpoints you need. Use `exclude` to remove noisy/admin endpoints.

### Definition Overrides
```yaml
definition_overrides:
  namespace: "integrations.harfanglab"    # Prefix for action names
  display_group: "HarfangLab EDR"         # UI grouping
  author: "adrojis"
  doc_url_prefix: "https://docs.harfanglab.io/api"
```

Per-action overrides:
```yaml
  name: "get_alert"           # Override action name
  title: "Get Alert Details"  # Human-readable title
  description: "..."          # Custom description
  deprecated: "Use v2"        # Mark as deprecated
```

### Authentication
```yaml
auth:
  secrets:
    - name: "harfanglab"
      keys: ["API_KEY", "BASE_URL"]
  injection:
    args:
      headers:
        Authorization: "Token ${{ SECRETS.harfanglab.API_KEY }}"
  expects:
    base_url:
      type: "str"
      description: "HarfangLab API base URL"
      default: "${{ SECRETS.harfanglab.BASE_URL }}"
```

**Auth patterns by vendor:**

| Vendor | Header | Format |
|--------|--------|--------|
| Generic API Key | `Authorization` | `ApiKey ${{ SECRETS.x.API_KEY }}` |
| Bearer Token | `Authorization` | `Bearer ${{ SECRETS.x.TOKEN }}` |
| Basic Auth | `Authorization` | `Basic ${{ FN.to_base64(SECRETS.x.USER + ":" + SECRETS.x.PASS) }}` |
| Custom Header | `X-Api-Key` | `${{ SECRETS.x.API_KEY }}` |
| Query Param | `params.api_key` | `${{ SECRETS.x.API_KEY }}` |

### Output Organization
```yaml
use_namespace_directories: true   # Subdirectories per namespace
```

## Path B: Manual Templates (Real Example — HarfangLab EDR)

HarfangLab has NO OpenAPI spec. We built 17 manual templates by researching their API from FortiSOAR connectors, Cortex XSOAR integrations, and SDK docs.

### API Details
- **Auth**: `Authorization: Token <API_KEY>`, base URL `https://<instance>:8443`
- **Sources used**: FortiSOAR connector, Cortex XSOAR integration, SDK

### Template structure (example: get_alert.yaml)
```yaml
type: action
definition:
  name: get_alert
  namespace: integrations.harfanglab
  title: Get Alert Details
  description: Retrieve detailed information about a specific alert
  display_group: HarfangLab EDR
  expects:
    alert_id:
      type: str
      description: Alert ID to retrieve
  secrets:
    - name: harfanglab
      keys: ["API_KEY", "BASE_URL"]
  steps:
    - ref: get_alert
      action: core.http_request
      args:
        method: GET
        url: "${{ SECRETS.harfanglab.BASE_URL }}/api/data/alert/alert/Alert/${{ inputs.alert_id }}"
        headers:
          Authorization: "Token ${{ SECRETS.harfanglab.API_KEY }}"
  returns: ${{ steps.get_alert.result }}
```

### Workflow built with these templates
**HarfangLab Alert Triage** (`wf_4PnQXYJQ65tDxRAI2pWxUo`) — 5 actions, fan-out/fan-in:
```
Trigger (500, 0) -> List Recent Alerts (500, 300)
                         |              |
           Get Alert Details    Search Affected Endpoint
           (200, 500)           (800, 500)
                         |              |
                    Create Triage Case (500, 700)
                              |
                    Update Alert Status (500, 900)
```

### Lessons from HarfangLab
1. **Research multiple sources** — no single source had all endpoints documented
2. **Start with core actions** — alerts, threats, IOCs, jobs (don't try to cover everything)
3. **Verify field names on live instance** — some field names from docs don't match reality
4. **Position nodes correctly** — fan-out needs 600px horizontal spacing, 200px vertical

## Path A: Step-by-Step

### 1. Find the OpenAPI Spec
- Check vendor docs for `/openapi.json` or `/swagger.json`
- Search GitHub: `repo:vendor/product openapi.json`
- Try: `https://<api-url>/openapi.json`, `https://<api-url>/v3/api-docs`
- Some vendors publish specs on SwaggerHub

### 2. Write the Config YAML
```yaml
# config-harfanglab.yaml
endpoints:
  include:
    like:
      - "/api/data/alert/alert/Alert*"
      - "/api/data/threat/ThreatIntel*"
      - "/api/data/Job*"
  exclude:
    like:
      - "/api/v1/admin/*"

definition_overrides:
  namespace: "integrations.harfanglab"
  display_group: "HarfangLab EDR"
  author: "adrojis"

auth:
  secrets:
    - name: "harfanglab"
      keys: ["API_KEY", "BASE_URL"]
  injection:
    args:
      headers:
        Authorization: "Token ${{ SECRETS.harfanglab.API_KEY }}"
  expects:
    base_url:
      type: "str"
      description: "HarfangLab instance URL"
      default: "${{ SECRETS.harfanglab.BASE_URL }}"
```

### 3. Run the Converter
```bash
uv run scripts/openapi_to_template.py \
  --input api-specs/harfanglab-openapi.json \
  --output-dir generated_templates/harfanglab \
  --config config-harfanglab.yaml
```

### 4. Review Generated Templates
- Check each YAML file in the output directory
- Verify action names, input schemas, HTTP methods
- Fix any `base_url` references if needed

### 5. Deploy to Tracecat
Option A: Copy templates to the local registry (`custom_registry/`)
Option B: Use the Tracecat API to register templates

### 6. Create the Secret
Use the MCP tool:
```
tracecat_create_secret(name="harfanglab", keys=["API_KEY", "BASE_URL"], values=["your-key", "https://your-instance.harfanglab.io"])
```

## Common Mistakes

### 1. Wrong OpenAPI version
OAS 2.0 (Swagger) won't work. Convert first via https://converter.swagger.io/

### 2. Too many endpoints
Don't convert the entire spec. Use `include.like` to select only what you need. Most APIs have 100+ endpoints but you only need 10-20.

### 3. Missing auth injection
Without `auth.injection`, generated templates won't authenticate. Always define the auth section.

### 4. Wrong secret key names
Secret keys must match EXACTLY what's in `injection.args.headers`. If you write `${{ SECRETS.harfanglab.API_KEY }}`, the secret must have a key named `API_KEY`.

### 5. Forgetting `base_url` in expects
Most APIs need a configurable base URL. Always add it in `auth.expects` with a default pointing to the secret.

### 6. Namespace collisions
If you generate templates for the same vendor twice, they'll overwrite. Use unique namespaces.

### 7. verify_ssl for on-premise APIs
On-prem instances often have self-signed certs. Add `verify_ssl: false` in the HTTP request args of generated templates.

### 8. Pagination not handled
The converter generates one-shot requests. For list endpoints that return paginated results, you may need to manually add loop logic.

## Architecture Note

**You do NOT need separate MCP servers per integration.** The OpenAPI converter + Tracecat secrets system handles everything. Our MCP server is the "control plane" for managing workflows, secrets, and cases.

## Reference
- Docs: https://docs.tracecat.com/integrations/openapi-converter
- Endpoint filtering: https://docs.tracecat.com/integrations/openapi-converter#endpoints-filtering
- Memory: `memory/openapi-converter.md`
- Memory: `memory/harfanglab-integration.md`
