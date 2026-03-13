---
name: tracecat-integration-expert
description: Activate when users configure external tool integrations (VirusTotal, CrowdStrike, Slack, Splunk, etc.) in Tracecat workflows
---

# Tracecat Integration Expert

You are an expert at configuring and using Tracecat integrations with external security tools and services.

## Integration Naming Convention

All integrations follow: `tools.<integration>.<action>`

## Common Integrations

### Threat Intelligence
| Integration | Actions | Secret Required |
|------------|---------|-----------------|
| VirusTotal | `analyze_url`, `analyze_hash`, `get_report` | `virustotal` (api_key) |
| AbuseIPDB | `check_ip`, `report_ip` | `abuseipdb` (api_key) |
| GreyNoise | `check_ip`, `query` | `greynoise` (api_key) |
| Shodan | `search`, `host_info` | `shodan` (api_key) |
| AlienVault OTX | `get_indicators` | `otx` (api_key) |

### EDR / Endpoint
| Integration | Actions | Secret Required |
|------------|---------|-----------------|
| CrowdStrike | `contain_host`, `lift_containment`, `search_detections` | `crowdstrike` (client_id, client_secret) |
| SentinelOne | `isolate_agent`, `get_threats` | `sentinelone` (api_key, url) |
| Microsoft Defender | `isolate_machine`, `get_alerts` | `msdefender` (tenant_id, client_id, client_secret) |

### SIEM
| Integration | Actions | Secret Required |
|------------|---------|-----------------|
| Splunk | `search`, `create_alert` | `splunk` (token, url) |
| Elastic | `search`, `get_alerts` | `elastic` (api_key, url) |

### Communication
| Integration | Actions | Secret Required |
|------------|---------|-----------------|
| Slack | `post_message`, `create_channel` | `slack` (bot_token) |
| PagerDuty | `create_incident`, `acknowledge` | `pagerduty` (api_key) |
| Email (SMTP) | `send_email` | `smtp` (host, port, user, password) |

### Identity
| Integration | Actions | Secret Required |
|------------|---------|-----------------|
| Okta | `suspend_user`, `reset_password` | `okta` (api_key, domain) |
| Azure AD | `disable_user`, `revoke_sessions` | `azuread` (tenant_id, client_id, client_secret) |

## Setting Up an Integration

### 1. Create the secret
```
Use tracecat_create_secret with:
  name: "virustotal"
  type: "custom"
  keys: [{ key: "api_key", value: "YOUR_API_KEY" }]
```

### 2. Use in workflow YAML
```yaml
- ref: check_hash
  action: tools.virustotal.analyze_hash
  args:
    hash: ${{ TRIGGER.data.file_hash }}
```

The secret is automatically resolved by Tracecat based on the integration name.

## Custom Integrations

For services without built-in integration, use `core.http.request`:

```yaml
- ref: custom_api_call
  action: core.http.request
  args:
    method: POST
    url: https://api.example.com/v1/endpoint
    headers:
      Authorization: "Bearer ${{ SECRETS.custom_api.token }}"
      Content-Type: application/json
    payload:
      data: ${{ TRIGGER.data }}
```

## Best Practices

1. **Secret naming** — Use the integration name as the secret name
2. **Least privilege** — Use API keys with minimum required permissions
3. **Rate limiting** — Add delays between bulk API calls
4. **Error handling** — Always handle API errors gracefully in workflows
5. **Testing** — Test integrations with non-destructive actions first

## Related Skills
- **tracecat-secrets-integrations** — Detailed secret configuration and integration setup (replaces this skill for secrets)
- **tracecat-action-configuration** — Action types and input configuration
- **tracecat-mcp-tools-expert** — MCP tool reference for secrets and actions
- **tracecat-workflow-patterns** — Workflow design patterns using integrations
- **tracecat-yaml-syntax** — YAML syntax for integration inputs
- **tracecat-validation-debug** — Debug integration errors (HTTP 4xx/5xx)
- **tracecat-code-python** — Custom integrations via Python scripts

## Reference Files
- [Common Mistakes](./COMMON_MISTAKES.md)
- [Examples](./EXAMPLES.md)
- [README](./README.md)
