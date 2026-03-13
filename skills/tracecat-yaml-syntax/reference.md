# Tracecat YAML Reference

## Complete Workflow Example

```yaml
definition:
  title: Alert Triage Pipeline
  description: Automatically triage and enrich incoming security alerts
  entrypoint:
    ref: receive_alert
  triggers:
    - type: webhook
      ref: webhook_trigger

  actions:
    - ref: receive_alert
      action: core.transform.reshape
      args:
        value:
          alert_type: ${{ TRIGGER.data.type }}
          source_ip: ${{ TRIGGER.data.src_ip }}
          severity: ${{ TRIGGER.data.severity }}

    - ref: enrich_ip
      action: tools.abuseipdb.check_ip
      args:
        ip: ${{ ACTIONS.receive_alert.result.source_ip }}
      depends_on:
        - receive_alert

    - ref: enrich_vt
      action: tools.virustotal.analyze_url
      args:
        url: ${{ TRIGGER.data.url }}
      depends_on:
        - receive_alert

    - ref: score_risk
      action: core.transform.reshape
      args:
        value:
          risk_score: ${{ FN.math.add(ACTIONS.enrich_ip.result.abuse_confidence_score, 0) }}
          is_malicious: ${{ ACTIONS.enrich_vt.result.malicious > 0 }}
      depends_on:
        - enrich_ip
        - enrich_vt

    - ref: create_case
      action: core.http.request
      args:
        method: POST
        url: ${{ ENV.TRACECAT_API_URL }}/cases
        headers:
          Content-Type: application/json
        payload:
          workflow_id: ${{ ENV.WORKFLOW_ID }}
          case_title: "[ALERT] ${{ ACTIONS.receive_alert.result.alert_type }}"
          priority: ${{ FN.conditional(ACTIONS.score_risk.result.risk_score > 70, "critical", "medium") }}
          malice: ${{ FN.conditional(ACTIONS.score_risk.result.is_malicious, "malicious", "unknown") }}
          payload: ${{ ACTIONS.score_risk.result }}
      depends_on:
        - score_risk
      run_if: ${{ ACTIONS.score_risk.result.risk_score > 30 }}

    - ref: notify_slack
      action: tools.slack.post_message
      args:
        channel: "#security-alerts"
        text: "New alert: ${{ ACTIONS.receive_alert.result.alert_type }} from ${{ ACTIONS.receive_alert.result.source_ip }} (risk: ${{ ACTIONS.score_risk.result.risk_score }})"
      depends_on:
        - score_risk
      run_if: ${{ ACTIONS.score_risk.result.risk_score > 50 }}
```

## Expression Functions

| Category | Function | Example |
|----------|----------|---------|
| String | `FN.str.upper(val)` | `${{ FN.str.upper(TRIGGER.data.name) }}` |
| String | `FN.str.lower(val)` | `${{ FN.str.lower(TRIGGER.data.type) }}` |
| String | `FN.str.contains(str, sub)` | `${{ FN.str.contains(TRIGGER.data.msg, "error") }}` |
| Math | `FN.math.add(a, b)` | `${{ FN.math.add(ACTIONS.a.result, 10) }}` |
| Logic | `FN.conditional(cond, true_val, false_val)` | `${{ FN.conditional(score > 70, "high", "low") }}` |
| List | `FN.list.length(arr)` | `${{ FN.list.length(ACTIONS.a.result.items) }}` |
| JSON | `FN.json.dumps(obj)` | `${{ FN.json.dumps(TRIGGER.data) }}` |

## Trigger Types

| Type | Config | Use Case |
|------|--------|----------|
| `webhook` | URL auto-generated | External alert ingestion |
| `schedule` | Cron expression | Periodic tasks |
| `manual` | None | On-demand execution |
