# Examples — Integrations

## Example 1: VirusTotal IP Lookup

**Secret setup:**
```
tracecat_create_secret({
  name: "virustotal",
  keys: [{ key: "API_KEY", value: "your-vt-api-key" }]
})
```

**Action:**
```yaml
type: core.http_request
title: VirusTotal IP Check
inputs:
  url: https://www.virustotal.com/api/v3/ip_addresses/${{ TRIGGER.data.ip }}
  method: GET
  headers:
    x-apikey: ${{ SECRETS.virustotal.API_KEY }}
    Accept: application/json
```

**Access results:**
```yaml
# Malicious count
${{ ACTIONS.vt_check.result.data.attributes.last_analysis_stats.malicious }}
# Country
${{ ACTIONS.vt_check.result.data.attributes.country }}
```

---

## Example 2: CrowdStrike Detection Query

**Secret setup:**
```
tracecat_create_secret({
  name: "crowdstrike",
  keys: [
    { key: "CLIENT_ID", value: "xxx" },
    { key: "CLIENT_SECRET", value: "yyy" },
    { key: "BASE_URL", value: "https://api.crowdstrike.com" }
  ]
})
```

**Step 1 — Get OAuth token:**
```yaml
type: core.http_request
title: CrowdStrike Auth
inputs:
  url: ${{ SECRETS.crowdstrike.BASE_URL }}/oauth2/token
  method: POST
  headers:
    Content-Type: application/x-www-form-urlencoded
  payload:
    client_id: ${{ SECRETS.crowdstrike.CLIENT_ID }}
    client_secret: ${{ SECRETS.crowdstrike.CLIENT_SECRET }}
```

**Step 2 — Query detections:**
```yaml
type: core.http_request
title: Query Detections
inputs:
  url: ${{ SECRETS.crowdstrike.BASE_URL }}/detects/queries/detects/v1
  method: GET
  headers:
    Authorization: Bearer ${{ ACTIONS.cs_auth.result.access_token }}
  params:
    filter: "device.hostname:'WORKSTATION01'"
    limit: "10"
```

---

## Example 3: Splunk Search

**Secret setup:**
```
tracecat_create_secret({
  name: "splunk",
  keys: [
    { key: "BASE_URL", value: "https://splunk.company.com:8089" },
    { key: "TOKEN", value: "your-splunk-token" }
  ]
})
```

**Action:**
```yaml
type: core.http_request
title: Splunk Search
inputs:
  url: ${{ SECRETS.splunk.BASE_URL }}/services/search/v2/jobs/export
  method: POST
  headers:
    Authorization: Bearer ${{ SECRETS.splunk.TOKEN }}
    Content-Type: application/x-www-form-urlencoded
  payload:
    search: "search index=main src_ip=${{ TRIGGER.data.src_ip }} earliest=-24h | stats count by dest_port"
    output_mode: json
```

---

## Example 4: Slack Notification

**Secret setup:**
```
tracecat_create_secret({
  name: "slack",
  keys: [{ key: "BOT_TOKEN", value: "xoxb-your-token" }]
})
```

**Action:**
```yaml
type: core.http_request
title: Notify SOC Channel
inputs:
  url: https://slack.com/api/chat.postMessage
  method: POST
  headers:
    Authorization: Bearer ${{ SECRETS.slack.BOT_TOKEN }}
    Content-Type: application/json
  payload:
    channel: "#soc-alerts"
    blocks:
      - type: header
        text:
          type: plain_text
          text: "Security Alert: ${{ ACTIONS.parse.result.title }}"
      - type: section
        fields:
          - type: mrkdwn
            text: "*Source IP:*\n${{ ACTIONS.parse.result.src_ip }}"
          - type: mrkdwn
            text: "*Severity:*\n${{ ACTIONS.score.result.priority }}"
          - type: mrkdwn
            text: "*VT Score:*\n${{ ACTIONS.enrich.result.vt_malicious }}"
```

---

## Example 5: Microsoft Defender — Get Alerts

**Secret setup:**
```
tracecat_create_secret({
  name: "defender",
  keys: [
    { key: "TENANT_ID", value: "xxx" },
    { key: "CLIENT_ID", value: "yyy" },
    { key: "CLIENT_SECRET", value: "zzz" }
  ]
})
```

**Step 1 — OAuth token:**
```yaml
type: core.http_request
title: Defender Auth
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
```

**Step 2 — Get alerts:**
```yaml
type: core.http_request
title: Get Defender Alerts
inputs:
  url: https://api.securitycenter.microsoft.com/api/alerts
  method: GET
  headers:
    Authorization: Bearer ${{ ACTIONS.defender_auth.result.access_token }}
  params:
    $filter: "severity eq 'High' and status eq 'New'"
    $top: "20"
```

---

## Example 6: CrowdSec CTI Lookup

**Secret setup:**
```
tracecat_create_secret({
  name: "crowdsec",
  keys: [{ key: "API_TOKEN", value: "your-crowdsec-cti-api-key" }],
  description: "CrowdSec CTI API token"
})
```

**Action:**
```yaml
type: core.http_request
title: CrowdSec IP Reputation
inputs:
  url: https://cti.api.crowdsec.net/v2/smoke/${{ TRIGGER.data.ip_address }}
  method: GET
  headers:
    x-api-key: ${{ SECRETS.crowdsec.API_TOKEN }}
    Accept: application/json
```

**Access results:**
```yaml
# Reputation (benign / malicious / suspicious)
${{ ACTIONS.crowdsec_lookup.result.reputation }}
# Background noise score (0-10)
${{ ACTIONS.crowdsec_lookup.result.background_noise_score }}
# Attack details
${{ ACTIONS.crowdsec_lookup.result.attack_details }}
```

---

## Example 7: Wazuh API — List Outdated Agents

**Secret setup:**
```
tracecat_create_secret({
  name: "wazuh",
  keys: [
    { key: "API_KEY", value: "your-wazuh-api-token" },
    { key: "SERVER_URL", value: "https://wazuh-manager.internal:55000" }
  ],
  description: "Wazuh Manager API credentials"
})
```

**Action:**
```yaml
type: core.http_request
title: Get Outdated Agents
inputs:
  url: ${{ SECRETS.wazuh.SERVER_URL }}/agents/outdated
  method: GET
  headers:
    Authorization: Bearer ${{ SECRETS.wazuh.API_KEY }}
  verify_ssl: false
```

**Important:** Wazuh API typically uses self-signed certs. Always set `verify_ssl: false`.

---

## Example 8: MISP — Create Event

**Secret setup:**
```
tracecat_create_secret({
  name: "misp",
  keys: [
    { key: "API_TOKEN", value: "your-misp-automation-key" },
    { key: "SERVER_URL", value: "https://misp.internal" }
  ],
  description: "MISP threat sharing platform credentials"
})
```

**Action:**
```yaml
type: core.http_request
title: Create MISP Event
inputs:
  url: ${{ SECRETS.misp.SERVER_URL }}/events/add
  method: POST
  headers:
    Authorization: ${{ SECRETS.misp.API_TOKEN }}
    Content-Type: application/json
    Accept: application/json
  payload:
    Event:
      info: "Tracecat alert: ${{ ACTIONS.parse.result.alert_name }}"
      distribution: "0"
      threat_level_id: "2"
      analysis: "1"
      Attribute:
        - type: ip-src
          value: ${{ ACTIONS.parse.result.src_ip }}
          category: Network activity
          to_ids: true
  verify_ssl: false
```

**Important:** MISP uses `Authorization` header (not `x-api-key`). The payload structure nests under `Event`.
