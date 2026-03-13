# Tracecat Workflow Pattern Examples

## Example 1: Phishing Email Analysis

```yaml
definition:
  title: Phishing Email Analyzer
  description: Analyze reported phishing emails
  entrypoint:
    ref: parse_email

  triggers:
    - type: webhook
      ref: email_report

  actions:
    - ref: parse_email
      action: core.transform.reshape
      args:
        value:
          sender: ${{ TRIGGER.data.from }}
          subject: ${{ TRIGGER.data.subject }}
          urls: ${{ TRIGGER.data.extracted_urls }}
          attachments: ${{ TRIGGER.data.attachments }}

    - ref: check_sender
      action: tools.abuseipdb.check_ip
      args:
        ip: ${{ ACTIONS.parse_email.result.sender_ip }}
      depends_on: [parse_email]

    - ref: scan_urls
      action: tools.virustotal.analyze_url
      args:
        url: ${{ var.url }}
      for_each: ${{ ACTIONS.parse_email.result.urls }}
      depends_on: [parse_email]

    - ref: verdict
      action: core.transform.reshape
      args:
        value:
          is_phishing: ${{ ACTIONS.scan_urls.result.malicious > 0 }}
          confidence: ${{ ACTIONS.check_sender.result.abuse_confidence_score }}
      depends_on: [check_sender, scan_urls]

    - ref: create_case
      action: core.http.request
      args:
        method: POST
        url: ${{ ENV.TRACECAT_API_URL }}/cases
        payload:
          case_title: "[PHISH] ${{ ACTIONS.parse_email.result.subject }}"
          priority: ${{ FN.conditional(ACTIONS.verdict.result.is_phishing, "high", "medium") }}
          malice: ${{ FN.conditional(ACTIONS.verdict.result.is_phishing, "malicious", "unknown") }}
      depends_on: [verdict]
```

## Example 2: Failed Login Monitor

```yaml
definition:
  title: Failed Login Alert
  description: Alert on excessive failed logins
  entrypoint:
    ref: query_logs

  triggers:
    - type: schedule
      ref: every_5_min
      config:
        cron: "*/5 * * * *"

  actions:
    - ref: query_logs
      action: core.http.request
      args:
        method: POST
        url: ${{ ENV.SIEM_URL }}/api/search
        headers:
          Authorization: "Bearer ${{ SECRETS.siem.api_key }}"
        payload:
          query: "event.type:authentication AND event.outcome:failure | stats count by user.name"
          timeframe: "5m"

    - ref: filter_high_count
      action: core.transform.reshape
      args:
        value: ${{ ACTIONS.query_logs.result.results }}
      run_if: ${{ FN.list.length(ACTIONS.query_logs.result.results) > 0 }}
      depends_on: [query_logs]

    - ref: notify
      action: tools.slack.post_message
      args:
        channel: "#security-alerts"
        text: "Failed login threshold exceeded for users: ${{ FN.json.dumps(ACTIONS.filter_high_count.result) }}"
      depends_on: [filter_high_count]
```

## Example 3: Vulnerability Remediation Tracker

```yaml
definition:
  title: Vuln Remediation Tracker
  description: Track vulnerability remediation progress
  entrypoint:
    ref: get_vulns

  triggers:
    - type: schedule
      ref: daily
      config:
        cron: "0 8 * * *"

  actions:
    - ref: get_vulns
      action: core.http.request
      args:
        method: GET
        url: ${{ ENV.VULN_SCANNER_URL }}/api/v1/vulnerabilities
        headers:
          Authorization: "Bearer ${{ SECRETS.vuln_scanner.api_key }}"
        payload:
          severity: ["critical", "high"]
          status: "open"

    - ref: check_existing_cases
      action: core.http.request
      args:
        method: GET
        url: ${{ ENV.TRACECAT_API_URL }}/cases
        payload:
          status: ["new", "in_progress"]
      depends_on: [get_vulns]

    - ref: create_new_cases
      action: core.transform.reshape
      args:
        value:
          new_vulns: ${{ ACTIONS.get_vulns.result }}
          existing: ${{ ACTIONS.check_existing_cases.result }}
      depends_on: [check_existing_cases]

    - ref: send_report
      action: tools.slack.post_message
      args:
        channel: "#vuln-management"
        text: "Daily Vuln Report: ${{ FN.list.length(ACTIONS.get_vulns.result) }} open critical/high vulnerabilities"
      depends_on: [create_new_cases]
```

## Example 4: Child Workflow Pattern (Parent + Child)

**Parent workflow** — iterates over alerts and delegates to child:
```yaml
definition:
  title: Alert Processor (Parent)
  description: Process each SIEM alert via a child enrichment workflow

  triggers:
    - type: webhook
      ref: batch_alerts

  actions:
    - ref: process_alert
      action: core.workflow.execute
      args:
        workflow_id: wf_CHILD_WORKFLOW_ID
        payload:
          alert: ${{ var.alert }}
      control_flow:
        for_each: ${{ TRIGGER.data.alerts }}

    - ref: summary
      action: core.transform.reshape
      args:
        value:
          total_processed: ${{ FN.length(ACTIONS.process_alert.result) }}
          results: ${{ ACTIONS.process_alert.result }}
      depends_on: [process_alert]
```

**Child workflow** — enriches and creates case for a single alert:
```yaml
definition:
  title: Alert Enrichment (Child)
  description: Enrich a single alert and create a case

  triggers:
    - type: webhook
      ref: single_alert

  actions:
    - ref: enrich_ip
      action: core.http_request
      args:
        url: https://www.virustotal.com/api/v3/ip_addresses/${{ TRIGGER.data.alert.src_ip }}
        method: GET
        headers:
          x-apikey: ${{ SECRETS.virustotal.API_KEY }}

    - ref: create_case
      action: core.cases.create_case
      args:
        case_title: "[Alert] ${{ TRIGGER.data.alert.name }}"
        priority: ${{ TRIGGER.data.alert.priority }}
        status: new
      depends_on: [enrich_ip]
```

## Example 5: SIEM-to-Case Severity Mapping

```yaml
definition:
  title: SIEM Alert to Case Mapper
  description: Standardize SIEM severity to Tracecat case priority

  triggers:
    - type: webhook
      ref: siem_alert

  actions:
    - ref: map_severity
      action: core.transform.reshape
      args:
        value:
          original_severity: ${{ TRIGGER.data.severity }}
          priority: ${{ FN.conditional(TRIGGER.data.severity == "critical" || TRIGGER.data.severity == "CRITICAL", "critical", FN.conditional(TRIGGER.data.severity == "high" || TRIGGER.data.severity == "HIGH", "high", FN.conditional(TRIGGER.data.severity == "medium" || TRIGGER.data.severity == "MEDIUM", "medium", "low"))) }}
          malice: ${{ FN.conditional(TRIGGER.data.severity == "critical" || TRIGGER.data.severity == "high", "malicious", "unknown") }}

    - ref: create_case
      action: core.cases.create_case
      args:
        case_title: "[SIEM] ${{ TRIGGER.data.alert_name }}"
        priority: ${{ ACTIONS.map_severity.result.priority }}
        malice: ${{ ACTIONS.map_severity.result.malice }}
        status: new
        payload:
          source: ${{ TRIGGER.data.source }}
          original_severity: ${{ ACTIONS.map_severity.result.original_severity }}
      depends_on: [map_severity]
```
