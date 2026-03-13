# Examples — Secrets & Integrations

## Example 1: VirusTotal Setup

```
# Create secret
tracecat_create_secret({
  name: "virustotal",
  keys: [{ key: "API_KEY", value: "your-vt-api-key-here" }],
  description: "VirusTotal API key for threat intel lookups"
})

# Use in workflow action
inputs:
  url: https://www.virustotal.com/api/v3/ip_addresses/${{ TRIGGER.data.ip }}
  method: GET
  headers:
    x-apikey: ${{ SECRETS.virustotal.API_KEY }}
```

---

## Example 2: CrowdStrike Setup (Multi-Key)

```
# Create secret with multiple keys
tracecat_create_secret({
  name: "crowdstrike",
  keys: [
    { key: "CLIENT_ID", value: "your-client-id" },
    { key: "CLIENT_SECRET", value: "your-client-secret" },
    { key: "BASE_URL", value: "https://api.crowdstrike.com" }
  ],
  description: "CrowdStrike Falcon API credentials"
})

# Use in OAuth token request
inputs:
  url: ${{ SECRETS.crowdstrike.BASE_URL }}/oauth2/token
  method: POST
  headers:
    Content-Type: application/x-www-form-urlencoded
  payload:
    client_id: ${{ SECRETS.crowdstrike.CLIENT_ID }}
    client_secret: ${{ SECRETS.crowdstrike.CLIENT_SECRET }}
```

---

## Example 3: Slack Bot Setup

```
# Create secret
tracecat_create_secret({
  name: "slack",
  keys: [{ key: "BOT_TOKEN", value: "xoxb-your-slack-bot-token" }],
  description: "Slack bot token for SOC notifications"
})

# Use in notification action
inputs:
  url: https://slack.com/api/chat.postMessage
  method: POST
  headers:
    Authorization: Bearer ${{ SECRETS.slack.BOT_TOKEN }}
    Content-Type: application/json
  payload:
    channel: "#soc-alerts"
    text: "Alert: ${{ ACTIONS.parse.result.title }}"
```

---

## Example 4: Splunk Setup

```
# Create secret
tracecat_create_secret({
  name: "splunk",
  keys: [
    { key: "BASE_URL", value: "https://splunk.company.com:8089" },
    { key: "TOKEN", value: "your-splunk-auth-token" }
  ],
  description: "Splunk Enterprise SIEM credentials"
})

# Use in search action
inputs:
  url: ${{ SECRETS.splunk.BASE_URL }}/services/search/v2/jobs/export
  method: POST
  headers:
    Authorization: Bearer ${{ SECRETS.splunk.TOKEN }}
  payload:
    search: "search index=main src_ip=${{ TRIGGER.data.ip }} earliest=-24h"
    output_mode: json
```

---

## Example 5: Microsoft Defender Setup

```
# Create secret
tracecat_create_secret({
  name: "defender",
  keys: [
    { key: "TENANT_ID", value: "your-azure-tenant-id" },
    { key: "CLIENT_ID", value: "your-app-client-id" },
    { key: "CLIENT_SECRET", value: "your-app-client-secret" }
  ],
  description: "Microsoft Defender for Endpoint API credentials"
})

# Step 1: Get OAuth token
inputs:
  url: https://login.microsoftonline.com/${{ SECRETS.defender.TENANT_ID }}/oauth2/v2.0/token
  method: POST
  headers:
    Content-Type: application/x-www-form-urlencoded
  payload:
    grant_type: client_credentials
    client_id: ${{ SECRETS.defender.CLIENT_ID }}
    client_secret: ${{ SECRETS.defender.CLIENT_SECRET }}
    scope: https://api.securitycenter.microsoft.com/.default

# Step 2: Use token
inputs:
  url: https://api.securitycenter.microsoft.com/api/alerts
  method: GET
  headers:
    Authorization: Bearer ${{ ACTIONS.defender_auth.result.access_token }}
```

---

## Example 6: Secret Management Operations

```
# Search secrets
tracecat_search_secrets({ name: "viru" })
# → Lists secrets matching "viru"

# Get secret metadata
tracecat_get_secret({ secret_name: "virustotal" })
# → Returns ID, name, type, description (NOT the actual key values)

# Update secret keys (rotate API key)
tracecat_update_secret({
  secret_id: "uuid-from-get-secret",
  keys: [{ key: "API_KEY", value: "new-rotated-key" }]
})

# Delete old secret
tracecat_delete_secret({ secret_id: "uuid-to-delete" })
```

---

## Example 7: Using Secrets in Python Scripts

```yaml
type: core.script.run_python
inputs:
  script: |
    import requests

    def main(api_key, base_url, target_ip):
        resp = requests.get(
            f"{base_url}/api/v3/ip_addresses/{target_ip}",
            headers={"x-apikey": api_key},
            timeout=30
        )
        resp.raise_for_status()
        return resp.json()
  inputs:
    api_key: ${{ SECRETS.virustotal.API_KEY }}
    base_url: "https://www.virustotal.com"
    target_ip: ${{ TRIGGER.data.ip }}
  allow_network: true
  dependencies:
    - requests
```

---

## Example 8: CrowdSec Setup

```
# Create secret
tracecat_create_secret({
  name: "crowdsec",
  keys: [{ key: "API_TOKEN", value: "your-crowdsec-cti-api-key" }],
  description: "CrowdSec CTI API token for IP reputation lookups"
})

# Use in IP lookup action
inputs:
  url: https://cti.api.crowdsec.net/v2/smoke/${{ TRIGGER.data.ip }}
  method: GET
  headers:
    x-api-key: ${{ SECRETS.crowdsec.API_TOKEN }}
```

---

## Example 9: MISP Setup

```
# Create secret
tracecat_create_secret({
  name: "misp",
  keys: [
    { key: "API_TOKEN", value: "your-misp-automation-key" },
    { key: "SERVER_URL", value: "https://misp.internal" }
  ],
  description: "MISP threat intelligence platform credentials"
})

# Use in event creation action
inputs:
  url: ${{ SECRETS.misp.SERVER_URL }}/events/add
  method: POST
  headers:
    Authorization: ${{ SECRETS.misp.API_TOKEN }}
    Content-Type: application/json
  verify_ssl: false
```

---

## Example 10: Wazuh Setup

```
# Create secret
tracecat_create_secret({
  name: "wazuh",
  keys: [
    { key: "API_KEY", value: "your-wazuh-api-token" },
    { key: "SERVER_URL", value: "https://wazuh-manager.internal:55000" }
  ],
  description: "Wazuh Manager API credentials"
})

# Use in agent query action
inputs:
  url: ${{ SECRETS.wazuh.SERVER_URL }}/agents
  method: GET
  headers:
    Authorization: Bearer ${{ SECRETS.wazuh.API_KEY }}
  verify_ssl: false
```

**Important:** Wazuh uses `Bearer` prefix. MISP uses the raw token without prefix.
