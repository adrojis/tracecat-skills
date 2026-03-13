---
name: tracecat-secrets-integrations
description: Activate when users configure secrets, connect integrations, set up Splunk, CrowdStrike, Okta, Wazuh, Slack, Jira, VirusTotal, GLIMPS, Microsoft Defender, or any third-party tool with Tracecat
---

# Tracecat Secrets & Integrations Expert

You are an expert at configuring secrets and connecting external tools with Tracecat.

## Secret Management

### Creating Secrets (MCP)
```
tracecat_create_secret:
  name: "virustotal"
  type: "custom"
  keys:
    - { key: "API_KEY", value: "your-api-key-here" }
```

### Secret Types
| Type | Use Case |
|------|----------|
| `custom` | Any key-value pairs (default) |
| `token` | Single bearer/API token |
| `oauth2` | OAuth2 credentials |
| `ssh` | SSH key pairs |

### Accessing Secrets in Workflows
```yaml
# In action inputs
headers:
  Authorization: "Bearer ${{ SECRETS.my_service.API_KEY }}"

# In Python scripts (pass via inputs, never access directly)
inputs:
  api_key: ${{ SECRETS.virustotal.API_KEY }}
```

### Secret Naming Convention
Use the integration name as the secret name for auto-resolution:
- `virustotal` for VirusTotal
- `crowdstrike` for CrowdStrike
- `splunk` for Splunk
- `slack` for Slack

## Integration Reference

### Threat Intelligence

#### VirusTotal
```
Secret: virustotal
Keys: API_KEY
```
```yaml
# Via native integration
action: tools.virustotal.analyze_hash
args:
  hash: ${{ TRIGGER.data.file_hash }}

# Via HTTP (API v3)
action: core.http_request
args:
  url: https://www.virustotal.com/api/v3/files/${{ TRIGGER.data.hash }}
  method: GET
  headers:
    x-apikey: ${{ SECRETS.virustotal.API_KEY }}
```

#### AbuseIPDB
```
Secret: abuseipdb
Keys: API_KEY
```
```yaml
action: core.http_request
args:
  url: https://api.abuseipdb.com/api/v2/check
  method: GET
  headers:
    Key: ${{ SECRETS.abuseipdb.API_KEY }}
  params:
    ipAddress: ${{ TRIGGER.data.ip }}
    maxAgeInDays: "90"
```

#### GreyNoise
```
Secret: greynoise
Keys: API_KEY
```

#### Shodan
```
Secret: shodan
Keys: API_KEY
```

### EDR / Endpoint

#### CrowdStrike Falcon
```
Secret: crowdstrike
Keys: CLIENT_ID, CLIENT_SECRET
```
```yaml
# Step 1: Get OAuth2 token
action: core.http_request
args:
  url: https://api.crowdstrike.com/oauth2/token
  method: POST
  headers:
    Content-Type: application/x-www-form-urlencoded
  payload:
    client_id: ${{ SECRETS.crowdstrike.CLIENT_ID }}
    client_secret: ${{ SECRETS.crowdstrike.CLIENT_SECRET }}

# Step 2: Use token in subsequent calls
action: core.http_request
args:
  url: https://api.crowdstrike.com/detects/queries/detects/v1
  method: GET
  headers:
    Authorization: "Bearer ${{ ACTIONS.get_cs_token.result.access_token }}"
```

#### Microsoft Defender for Endpoint
```
Secret: msdefender
Keys: TENANT_ID, CLIENT_ID, CLIENT_SECRET
```
```yaml
# Step 1: Get Azure AD token
action: core.http_request
args:
  url: https://login.microsoftonline.com/${{ SECRETS.msdefender.TENANT_ID }}/oauth2/v2.0/token
  method: POST
  headers:
    Content-Type: application/x-www-form-urlencoded
  payload:
    client_id: ${{ SECRETS.msdefender.CLIENT_ID }}
    client_secret: ${{ SECRETS.msdefender.CLIENT_SECRET }}
    scope: https://api.securitycenter.microsoft.com/.default
    grant_type: client_credentials
```

#### SentinelOne
```
Secret: sentinelone
Keys: API_KEY, BASE_URL
```

### SIEM

#### Splunk
```
Secret: splunk
Keys: TOKEN, BASE_URL
```
```yaml
action: core.http_request
args:
  url: ${{ SECRETS.splunk.BASE_URL }}/services/search/jobs
  method: POST
  headers:
    Authorization: "Bearer ${{ SECRETS.splunk.TOKEN }}"
    Content-Type: application/x-www-form-urlencoded
  payload:
    search: "search index=main sourcetype=syslog | head 100"
    output_mode: json
```

#### Elastic / OpenSearch
```
Secret: elastic
Keys: API_KEY, BASE_URL
```

#### Wazuh
```
Secret: wazuh
Keys: USER, PASSWORD, BASE_URL
```
```yaml
# Step 1: Authenticate
action: core.http_request
args:
  url: ${{ SECRETS.wazuh.BASE_URL }}/security/user/authenticate
  method: POST
  headers:
    Content-Type: application/json
    Authorization: "Basic ${{ FN.base64_encode(SECRETS.wazuh.USER + ':' + SECRETS.wazuh.PASSWORD) }}"
```

### Communication

#### Slack
```
Secret: slack
Keys: BOT_TOKEN
```
```yaml
action: core.http_request
args:
  url: https://slack.com/api/chat.postMessage
  method: POST
  headers:
    Authorization: "Bearer ${{ SECRETS.slack.BOT_TOKEN }}"
    Content-Type: application/json
  payload:
    channel: "#security-alerts"
    text: "Alert: ${{ TRIGGER.data.alert_name }}"
```

#### Microsoft Teams (via Webhook)
```
Secret: teams
Keys: WEBHOOK_URL
```
```yaml
action: core.http_request
args:
  url: ${{ SECRETS.teams.WEBHOOK_URL }}
  method: POST
  headers:
    Content-Type: application/json
  payload:
    text: "Security alert: ${{ TRIGGER.data.message }}"
```

#### PagerDuty
```
Secret: pagerduty
Keys: API_KEY, ROUTING_KEY
```

#### Email (SMTP)
```
Secret: smtp
Keys: HOST, PORT, USER, PASSWORD
```

### Ticketing

#### Jira
```
Secret: jira
Keys: API_TOKEN, EMAIL, BASE_URL
```
```yaml
action: core.http_request
args:
  url: ${{ SECRETS.jira.BASE_URL }}/rest/api/3/issue
  method: POST
  headers:
    Authorization: "Basic ${{ FN.base64_encode(SECRETS.jira.EMAIL + ':' + SECRETS.jira.API_TOKEN) }}"
    Content-Type: application/json
  payload:
    fields:
      project:
        key: SEC
      summary: ${{ TRIGGER.data.alert_title }}
      issuetype:
        name: Task
      description:
        type: doc
        version: 1
        content:
          - type: paragraph
            content:
              - type: text
                text: ${{ TRIGGER.data.description }}
```

#### ServiceNow
```
Secret: servicenow
Keys: INSTANCE_URL, USER, PASSWORD
```

### Identity & Access

#### Okta
```
Secret: okta
Keys: API_KEY, DOMAIN
```
```yaml
# Suspend a user
action: core.http_request
args:
  url: https://${{ SECRETS.okta.DOMAIN }}/api/v1/users/${{ TRIGGER.data.user_id }}/lifecycle/suspend
  method: POST
  headers:
    Authorization: "SSWS ${{ SECRETS.okta.API_KEY }}"
```

#### Azure AD / Entra ID
```
Secret: azuread
Keys: TENANT_ID, CLIENT_ID, CLIENT_SECRET
```

### File Analysis

#### GLIMPS Malware
```
Secret: glimps
Keys: API_KEY, BASE_URL
```
```yaml
action: core.http_request
args:
  url: ${{ SECRETS.glimps.BASE_URL }}/submit
  method: POST
  headers:
    Authorization: "Bearer ${{ SECRETS.glimps.API_KEY }}"
```

## MCP Tools for Secrets

| Tool | Usage |
|------|-------|
| `tracecat_create_secret` | Create new secret with key-value pairs |
| `tracecat_get_secret` | Get secret metadata (NOT decrypted values) |
| `tracecat_update_secret` | Update secret keys or description |
| `tracecat_delete_secret` | Permanently delete a secret |
| `tracecat_search_secrets` | Search secrets by name |

## Best Practices

1. **Naming** — Use the integration name as the secret name
2. **Key naming** — Use UPPERCASE for key names (API_KEY, CLIENT_ID)
3. **Least privilege** — Use API keys with minimum required permissions
4. **Rotation** — Rotate secrets regularly via `tracecat_update_secret`
5. **No hardcoding** — Never put credentials in action inputs directly
6. **Test first** — Test integrations with non-destructive read-only actions before containment/blocking
7. **Rate limiting** — Add `start_delay` or batch processing for high-volume API calls
8. **Error handling** — Use error edges to handle API failures (401, 403, 429)

## Common Mistakes

| Mistake | Correct |
|---------|---------|
| `${{ SECRETS.vt.api_key }}` (lowercase) | `${{ SECRETS.virustotal.API_KEY }}` |
| Hardcoded API key in URL | Use `${{ SECRETS.name.KEY }}` |
| Accessing secrets in Python directly | Pass via `inputs` field |
| Creating duplicate secret names | Check with `tracecat_search_secrets` first |
| Forgetting OAuth2 token step | CrowdStrike/Defender need token exchange first |

## Related Skills
- **tracecat-action-configuration** — Action types and input configuration
- **tracecat-mcp-tools-expert** — MCP tools for secret operations
- **tracecat-workflow-patterns** — Integration patterns in workflows
- **tracecat-code-python** — Custom integrations via Python scripts
- **tracecat-validation-debug** — Debug secret access errors

## Reference Files
- [Common Mistakes](./COMMON_MISTAKES.md)
- [Examples](./EXAMPLES.md)
- [README](./README.md)
