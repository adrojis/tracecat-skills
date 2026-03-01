---
name: tracecat-secrets-integrations
description: Tracecat integrations and secrets configuration expert. Use when connecting integrations, configuring secrets, setting up Splunk, CrowdStrike, Okta, Wazuh, Slack, Jira, VirusTotal, or any third-party tool with Tracecat.
---

# Tracecat Secrets & Integrations

## Secrets Syntax

```yaml
${{ SECRETS.<secret_name>.<key> }}
```

Secrets are encrypted at rest, workspace-scoped, and garbage-collected after action execution.

## Secret Types

| Type | Use case |
|------|----------|
| `custom` | API keys, tokens (default) |
| `ssh-key` | Git repo sync |
| `mtls` | Mutual TLS (`TLS_CERTIFICATE`, `TLS_PRIVATE_KEY`) |
| `ca_cert` | Custom CA (`CA_CERTIFICATE`) |

## OAuth Secrets

```yaml
# User token (authorization code flow)
${{ SECRETS.<provider>_oauth.<PREFIX>_USER_TOKEN }}

# Service token (client credentials flow)
${{ SECRETS.<provider>_oauth.<PREFIX>_SERVICE_TOKEN }}
```

## Workspace Variables (Base URLs)

Set once, used everywhere:
```
VARS.splunk.base_url = https://splunk.example.com:8089
VARS.okta.base_url = https://your-org.okta.com
VARS.jira.base_url = https://your-org.atlassian.net
VARS.sentinel_one.base_url = https://your-console.sentinelone.net
VARS.elasticsearch.base_url = https://elastic.example.com:9200
```

## Integration Reference (42 namespaces, 250+ actions)

### Security & Threat Intel

| Integration | Namespace | Secret Name | Keys | Key Actions |
|-------------|-----------|-------------|------|-------------|
| **CrowdStrike** | `tools.crowdstrike` | `crowdstrike` | `CROWDSTRIKE_CLIENT_ID`, `CROWDSTRIKE_CLIENT_SECRET` | `list_alerts`, `list_detects`, `list_cases` |
| **CrowdStrike FalconPy** | `tools.falconpy` | `crowdstrike` | (same) | `call_command` (any operation_id) |
| **SentinelOne** | `tools.sentinel_one` | `sentinel_one` | `SENTINELONE_API_KEY` | `list_threats` |
| **Wazuh** | `tools.wazuh` | `wazuh` | `WAZUH_API_TOKEN`, `WAZUH_API_URL` | `active_response`, `update_agents` |
| **CrowdSec** | `tools.crowdsec` | `crowdsec` | `CROWDSEC_API_KEY` | `lookup_ip_address` |
| **VirusTotal** | `tools.virustotal` | `virustotal` | `VIRUSTOTAL_API_KEY` | `lookup_domain`, `lookup_file_hash`, `lookup_ip_address`, `lookup_url` |
| **URLScan** | `tools.urlscan` | `urlscan` | `URLSCAN_API_KEY` | `lookup_url` |
| **IPInfo** | `tools.ipinfo` | `ipinfo` | `IPINFO_API_KEY` | `lookup_ip_address` |
| **ThreatStream** | `tools.threatstream` | `threatstream` | `THREATSTREAM_API_KEY`, `THREATSTREAM_USERNAME` | `lookup_domain`, `lookup_email`, `lookup_file_hash`, `lookup_ip_address`, `lookup_url` |
| **HIBP** | `tools.hibp` | `hibp` | `HIBP_API_KEY` | `check_email_breaches`, `check_email_pastes`, `get_all_breaches` |
| **Elastic Security** | `tools.elastic_security` | `elastic_security` | `ELASTIC_API_KEY` | `list_detection_signals` |
| **Datadog** | `tools.datadog` | `datadog` | `DATADOG_API_KEY`, `DATADOG_APP_KEY` | `list_security_signals` |
| **LeakCheck** | `tools.leakcheck` | `leakcheck` | `LEAKCHECK_API_KEY` | `search_email_leak`, `search_domain_leak` |
| **HackerOne** | `tools.hackerone` | `hackerone` | `HACKERONE_API_KEY`, `HACKERONE_API_NAME` | `get_programs`, `get_reports` |
| **PhishLabs** | `tools.phishlabs` | `phishlabs` | `PHISHLABS_API_KEY`, `PHISHLABS_API_SECRET` | `get_case_data`, `get_feed_data`, `get_threat_data` |

### Splunk (18 actions)

| Secret | Keys | Port |
|--------|------|------|
| `splunk` | `SPLUNK_API_KEY` | 8089 (REST API) |
| `splunk_hec` | `SPLUNK_HEC_TOKEN` | 8088 (HEC) |

**Search & Discovery** (6):
`search_events`, `discover_fields`, `list_indexes`, `list_sourcetypes`, `list_data_models`, `list_field_extractions`

**KV Store** (10):
`create_kv_collection`, `get_kv_collection`, `delete_kv_collection`, `list_kv_collections`, `add_kv_fields`, `create_kv_entry`, `get_kv_entry`, `update_kv_entry`, `delete_kv_entry`, `list_kv_entries`

KV Store query supports MongoDB-style filters:
```yaml
query:
  ip: {"$regex": "192.168.*"}
  severity: {"$gt": 5}
```

**HEC** (1): `submit_hec_event`
**CSV** (1): `upload_csv_to_kv_collection`

**Missing Splunk capabilities** (use `core.http_request`):

```yaml
# Saved searches
url: ${{ VARS.splunk.base_url }}/servicesNS/-/-/saved/searches
method: GET
headers:
  Authorization: Bearer ${{ SECRETS.splunk.SPLUNK_API_KEY }}

# Dispatch saved search
url: ${{ VARS.splunk.base_url }}/servicesNS/-/-/saved/searches/MySearch/dispatch
method: POST

# Notable events (Splunk ES)
url: ${{ VARS.splunk.base_url }}/services/notable_update
method: POST
payload:
  ruleUIDs: <rule_id>
  status: "2"
  comment: "Auto-triaged by Tracecat"

# Alerts
url: ${{ VARS.splunk.base_url }}/servicesNS/-/-/alerts/fired_alerts
method: GET

# Dashboards
url: ${{ VARS.splunk.base_url }}/servicesNS/-/-/data/ui/views
method: GET
```

### Identity & Access

| Integration | Secret | Keys | Key Actions |
|-------------|--------|------|-------------|
| **Okta** | `okta` | `OKTA_API_KEY` | `list_users`, `suspend_user`, `revoke_sessions`, `lookup_user_by_email`, `clear_user_sessions`, `get_user` + 13 more |
| **LDAP** | `ldap` | `LDAP_SERVER`, `LDAP_USERNAME`, `LDAP_PASSWORD`, `LDAP_BASE_DN` | `add_entry`, `delete_entry`, `modify_entry`, `search_entries` |

### Incident & IT Management

| Integration | Secret | Keys | Key Actions |
|-------------|--------|------|-------------|
| **Jira** | `jira` | `JIRA_API_KEY`, `JIRA_EMAIL` | `create_issue`, `get_issue`, `search_issues`, `update_issue_status`, `add_issue_comment`, `upload_attachment` + 9 more |
| **PagerDuty** | `pagerduty` | `PAGERDUTY_API_KEY` | `trigger_event`, `acknowledge_event`, `resolve_event`, `get_incidents` + 5 more |
| **Rootly** | `rootly` | `ROOTLY_API_KEY` | `create_incident`, `create_alert`, `list_incidents` + 4 more |
| **Zendesk** | `zendesk` | `ZENDESK_API_KEY`, `ZENDESK_EMAIL`, `ZENDESK_SUBDOMAIN` | `search_tickets`, `get_ticket`, `get_ticket_comments` + 4 more |

### Communication

| Integration | Secret | Keys | Key Actions |
|-------------|--------|------|-------------|
| **Slack** | `slack` | `SLACK_BOT_TOKEN` | `post_message`, `post_notification`, `post_update`, `ask_text_input`, `lookup_user_by_email`, `revoke_sessions` |
| **Slack SDK** | `slack` | (same) | `call_method`, `call_paginated_method` (any Slack API method) |

### Cloud & Infra

| Integration | Secret | Keys | Key Actions |
|-------------|--------|------|-------------|
| **AWS Boto3** | `aws` | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` | `call_api`, `call_paginated_api` (any AWS service) |
| **Amazon S3** | `aws` | (same) | `download_object`, `parse_uri` |
| **Ansible** | `ansible` | `ANSIBLE_PRIVATE_KEY` | `run_playbook` |
| **MongoDB** | `pymongo` | `PYMONGO_CONNECTION_STRING` | `execute_operation` |

### AI/LLM

| Action | Provider support |
|--------|------------------|
| `ai.action` | OpenAI, Anthropic, Bedrock, Gemini, Ollama |
| `ai.agent` | Same + tool calling |

Configure via secrets or env vars:
- `OPENAI_API_KEY` for OpenAI (and Ollama via `OPENAI_BASE_URL`)
- `ANTHROPIC_API_KEY` for Claude
- AWS credentials for Bedrock
- `GOOGLE_API_KEY` for Gemini

## IOC Extraction Functions (built-in)

No integration needed -- these are `FN.*` functions:

```yaml
${{ FN.extract_ipv4(TRIGGER.raw_text) }}
${{ FN.extract_sha256(TRIGGER.raw_text) }}
${{ FN.extract_cves(TRIGGER.raw_text) }}
${{ FN.extract_domains(TRIGGER.raw_text, true) }}  # include defanged
${{ FN.extract_urls(TRIGGER.raw_text) }}
${{ FN.extract_emails(TRIGGER.raw_text) }}
${{ FN.extract_md5(TRIGGER.raw_text) }}
${{ FN.extract_mac(TRIGGER.raw_text) }}
${{ FN.extract_asns(TRIGGER.raw_text) }}
${{ FN.ipv4_is_public("1.2.3.4") }}
${{ FN.ipv4_in_subnet("10.0.0.5", "10.0.0.0/8") }}
```

## Custom Integrations

Two approaches:

### YAML Template (simple REST APIs)
```yaml
type: action
definition:
  namespace: custom.mycompany.my_tool
  name: get_data
  secrets:
    - name: my_tool
      keys: ["MY_TOOL_API_KEY"]
  expects:
    query:
      type: str
      description: Search query
  steps:
    - ref: call_api
      action: core.http_request
      args:
        url: https://api.mytool.com/search
        method: GET
        headers:
          Authorization: Bearer ${{ SECRETS.my_tool.MY_TOOL_API_KEY }}
        params:
          q: ${{ inputs.query }}
  returns: ${{ steps.call_api.result }}
```

### Python UDF (complex logic)
```python
from tracecat_registry import registry, RegistrySecret, secrets

@registry.register(
    default_title="My Tool Query",
    namespace="custom.mycompany.my_tool",
    secrets=[RegistrySecret(name="my_tool", keys=["MY_TOOL_API_KEY"])]
)
def query_data(query: str):
    api_key = secrets.get("MY_TOOL_API_KEY")
    # complex logic here
    return {"results": [...]}
```

## Multi-Tenant Secrets

Same workflow, different credentials per tenant:

```yaml
- ref: run_per_tenant
  action: core.workflow.execute
  for_each: ${{ for var.tenant in ["tenant_a", "tenant_b"] }}
  args:
    workflow_alias: check_compliance
    trigger_inputs:
      tenant: ${{ var.tenant }}
    environment: ${{ var.tenant }}
```

Each tenant has its own secret set under a different environment key.