---
name: tracecat-workflow-patterns
description: Proven workflow patterns for Tracecat SOAR. Use when building workflows, designing automation, creating alert triage, enrichment pipelines, compliance checks, Splunk integrations, or planning workflow architecture.
---

# Tracecat Workflow Patterns

## Pattern 1: Alert Triage (SIEM -> Enrich -> Case)

Trigger: webhook from SIEM (Splunk, Wazuh, Elastic)

```yaml
actions:
  - ref: receive_alert
    action: core.transform.reshape
    args:
      value:
        alert_id: ${{ TRIGGER.alert_id }}
        severity: ${{ TRIGGER.severity }}
        src_ip: ${{ TRIGGER.src_ip }}

  - ref: enrich_ip
    action: tools.virustotal.lookup_ip_address
    depends_on: [receive_alert]
    args:
      ip_address: ${{ ACTIONS.receive_alert.result.src_ip }}

  - ref: check_public
    action: core.transform.reshape
    depends_on: [receive_alert]
    args:
      value: ${{ FN.ipv4_is_public(ACTIONS.receive_alert.result.src_ip) }}

  - ref: create_case
    action: core.cases.create_case
    depends_on: [enrich_ip]
    run_if: ${{ ACTIONS.check_public.result == true }}
    args:
      summary: "Alert: ${{ TRIGGER.alert_name }}"
      priority: high
      severity: ${{ TRIGGER.severity }}
      tags: ["auto-triage"]
```

## Pattern 2: Enrichment Pipeline (IOC -> Multi-source -> Store)

```yaml
actions:
  - ref: extract_iocs
    action: core.transform.reshape
    args:
      value:
        ips: ${{ FN.extract_ipv4(TRIGGER.raw_text) }}
        hashes: ${{ FN.extract_sha256(TRIGGER.raw_text) }}
        urls: ${{ FN.extract_urls(TRIGGER.raw_text) }}

  - ref: vt_lookup
    action: tools.virustotal.lookup_ip_address
    depends_on: [extract_iocs]
    for_each: ${{ for var.ip in ACTIONS.extract_iocs.result.ips }}
    args:
      ip_address: ${{ var.ip }}

  - ref: store_results
    action: core.table.insert_row
    depends_on: [vt_lookup]
    args:
      table: ioc_enrichments
      row_data:
        iocs: ${{ ACTIONS.extract_iocs.result }}
        vt_results: ${{ ACTIONS.vt_lookup.result }}
      upsert: true
```

## Pattern 3: Compliance Check (Okta -> Filter -> Alert)

```yaml
actions:
  - ref: list_users
    action: tools.okta.list_users
    args: {}

  - ref: filter_no_mfa
    action: core.transform.filter
    depends_on: [list_users]
    args:
      items: ${{ ACTIONS.list_users.result[*] }}
      python_lambda: >-
        lambda u: not u.get('mfa_enrolled', False)

  - ref: notify_slack
    action: tools.slack.post_message
    depends_on: [filter_no_mfa]
    run_if: ${{ FN.length(ACTIONS.filter_no_mfa.result) > 0 }}
    args:
      channel: "#compliance-alerts"
      text: "${{ FN.length(ACTIONS.filter_no_mfa.result) }} users without MFA"

  - ref: create_jira
    action: tools.jira.create_issue
    depends_on: [filter_no_mfa]
    run_if: ${{ FN.length(ACTIONS.filter_no_mfa.result) > 0 }}
    args:
      summary: "MFA compliance: ${{ FN.length(ACTIONS.filter_no_mfa.result) }} users non-compliant"
      priority_id: "2"
```

## Pattern 4: Splunk Alert Triage (Saved Search -> Enrich -> HEC writeback)

```yaml
triggers:
  - type: webhook
    ref: splunk_alert
    entrypoint: receive_splunk_alert

actions:
  - ref: receive_splunk_alert
    action: core.transform.reshape
    args:
      value:
        search_name: ${{ TRIGGER.search_name }}
        result: ${{ TRIGGER.result }}
        sid: ${{ TRIGGER.sid }}

  # Pull full results from Splunk (webhook only sends first row)
  - ref: get_full_results
    action: tools.splunk.search_events
    depends_on: [receive_splunk_alert]
    args:
      query: "search index=main | loadjob ${{ ACTIONS.receive_splunk_alert.result.sid }}"
      start_time: ${{ FN.to_isoformat(FN.now() - FN.hours(1)) }}
      end_time: ${{ FN.to_isoformat(FN.now()) }}
      limit: 100

  - ref: enrich_ips
    action: tools.virustotal.lookup_ip_address
    depends_on: [get_full_results]
    for_each: ${{ for var.event in ACTIONS.get_full_results.result.data.results }}
    args:
      ip_address: ${{ var.event.src_ip }}

  # Write enrichment results back to Splunk
  - ref: writeback_to_splunk
    action: tools.splunk.submit_hec_event
    depends_on: [enrich_ips]
    args:
      event:
        original_sid: ${{ ACTIONS.receive_splunk_alert.result.sid }}
        enrichment: ${{ ACTIONS.enrich_ips.result }}
        action_taken: "auto-enriched"
      source: tracecat_enrichment
      sourcetype: tracecat_action_log
      index: security_enrichment
```

## Pattern 5: Splunk Notable Events via HTTP (Missing Integration)

Use `core.http_request` for Splunk ES notable event management:

```yaml
  # Query notable events
  - ref: get_notables
    action: core.http_request
    args:
      url: ${{ VARS.splunk.base_url }}/services/search/jobs
      method: POST
      headers:
        Authorization: Bearer ${{ SECRETS.splunk.SPLUNK_API_KEY }}
      payload:
        search: "| search `notable` | head 50"
        exec_mode: oneshot
        output_mode: json
      verify_ssl: false

  # Update notable event status
  - ref: update_notable
    action: core.http_request
    args:
      url: ${{ VARS.splunk.base_url }}/services/notable_update
      method: POST
      headers:
        Authorization: Bearer ${{ SECRETS.splunk.SPLUNK_API_KEY }}
      payload:
        ruleUIDs: ${{ ACTIONS.get_notables.result.data.results[0].rule_id }}
        status: "2"
        comment: "Auto-triaged by Tracecat"
      verify_ssl: false
```

## Pattern 6: Splunk Saved Searches via HTTP (Missing Integration)

```yaml
  # List saved searches
  - ref: list_saved_searches
    action: core.http_request
    args:
      url: ${{ VARS.splunk.base_url }}/servicesNS/-/-/saved/searches
      method: GET
      headers:
        Authorization: Bearer ${{ SECRETS.splunk.SPLUNK_API_KEY }}
      params:
        output_mode: json
        count: 50
      verify_ssl: false

  # Dispatch a saved search
  - ref: dispatch_search
    action: core.http_request
    args:
      url: ${{ VARS.splunk.base_url }}/servicesNS/-/-/saved/searches/My%20Alert/dispatch
      method: POST
      headers:
        Authorization: Bearer ${{ SECRETS.splunk.SPLUNK_API_KEY }}
      params:
        output_mode: json
      verify_ssl: false
```

## Pattern 7: AI-SOAR Triage (Wazuh -> AI -> Act)

```yaml
actions:
  - ref: auth_wazuh
    action: core.http_request
    args:
      url: https://wazuh-manager:55000/security/user/authenticate
      method: POST
      auth:
        username: ${{ SECRETS.wazuh.WAZUH_USER }}
        password: ${{ SECRETS.wazuh.WAZUH_PASSWORD }}
      verify_ssl: false

  - ref: get_packages
    action: core.http_request
    depends_on: [auth_wazuh]
    args:
      url: https://wazuh-manager:55000/syscollector/001/packages
      method: GET
      headers:
        Authorization: Bearer ${{ ACTIONS.auth_wazuh.result.data.data.token }}
      verify_ssl: false

  - ref: ai_triage
    action: ai.action
    depends_on: [get_packages]
    args:
      user_prompt: |
        Analyze these packages for security risks. Return JSON with:
        - risk_score (1-10)
        - critical_packages (list)
        - recommendations (list)
        Packages: ${{ FN.serialize_json(ACTIONS.get_packages.result.data) }}
      model_name: gpt-4o-mini
      model_provider: openai

  - ref: notify
    action: tools.slack.post_message
    depends_on: [ai_triage]
    args:
      channel: "#soc-alerts"
      text: "AI Triage Result:\n${{ FN.prettify_json(ACTIONS.ai_triage.result) }}"
```

## Control Flow Quick Reference

| Feature | Syntax |
|---------|--------|
| Conditional | `run_if: ${{ ACTIONS.x.result.status == 200 }}` |
| Loop | `for_each: ${{ for var.item in ACTIONS.x.result[*] }}` |
| Loop variable | `${{ var.item.field }}` |
| Join any | `join_strategy: any` (run if ANY upstream completes) |
| Join all | `join_strategy: all` (wait for ALL upstreams) |
| Combined conditions | `${{ condition1 && condition2 }}` |
| Default value | `${{ ACTIONS.x.result.field \|\| "default" }}` |
| Null check | `${{ ACTIONS.x.result != None }}` |
| Start delay | `start_delay: 5.0` (seconds) |

## Workflow Architecture Best Practices

1. **Reshape = debug**: Use `core.transform.reshape` to rename/inspect data between actions
2. **Stop-check pattern**: Validate each step before proceeding (run_if on success)
3. **Error contract**: Python UDFs should return `{"success": bool, "error": str}`
4. **Dual notification**: Case comments (audit trail) + Slack (real-time)
5. **Lazy enrichment**: Pull data on-demand, not pre-cached
6. **Schema-constrained AI**: Force LLM to return structured JSON, not free text

## Programmatic Workflow Creation (API / MCP)

### Step-by-step creation sequence

```
1. create_workflow -> get workflow_id
2. create_action (x N) -> get action IDs and refs
3. update_action (x N) -> set inputs (YAML string)
4. Link nodes via Graph API (edges + positions)
5. deploy_workflow (commit)
6. Set workflow status to "online"
7. Activate webhook (set webhook status to "online")
8. Trigger via webhook URL
```

### Graph API: Linking nodes (edges)

Endpoint: `PATCH /workflows/{id}/graph?workspace_id=...`

```json
{
  "base_version": 1,
  "operations": [
    {
      "type": "add_edge",
      "payload": {
        "source_id": "trigger-{workflow_internal_uuid}",
        "source_type": "trigger",
        "target_id": "{first_action_uuid}"
      }
    },
    {
      "type": "add_edge",
      "payload": {
        "source_id": "{action_1_uuid}",
        "source_type": "udf",
        "target_id": "{action_2_uuid}"
      }
    }
  ]
}
```

**Key rules:**
- `base_version` must match current `graph_version` of the workflow
- Trigger `source_id` format: `trigger-{workflow_internal_uuid}` (found in `webhook.workflow_id`)
- Action edges use `source_type: "udf"`
- Trigger edge uses `source_type: "trigger"`

### Graph API: Positioning nodes visually

```json
{
  "base_version": 2,
  "operations": [
    {
      "type": "update_trigger_position",
      "payload": { "x": 250, "y": 0 }
    },
    {
      "type": "move_nodes",
      "payload": {
        "positions": [
          { "action_id": "{uuid}", "x": 250, "y": 350 },
          { "action_id": "{uuid}", "x": 250, "y": 550 }
        ]
      }
    }
  ]
}
```

**Available graph operations:** `add_node`, `update_node`, `delete_node`, `add_edge`, `delete_edge`, `move_nodes`, `update_trigger_position`, `update_viewport`

### Triggering a workflow

The `/workflows/{id}/execute` endpoint does NOT exist.
Use the **webhook URL** instead:

```
POST /api/webhooks/{wf_id}/{webhook_secret}
Content-Type: application/json
{}
```

The webhook must be set to `"online"` status first.