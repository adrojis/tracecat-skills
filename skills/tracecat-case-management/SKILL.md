---
name: tracecat-case-management
description: Tracecat case management expert. Use when creating cases, managing incidents, updating case status, adding comments, configuring custom fields, or building case automation workflows.
---

# Tracecat Case Management

## Case Lifecycle

```
new -> in_progress -> on_hold -> resolved -> closed
                  \-> other
```

## Case Fields

| Field | Type | Values |
|-------|------|--------|
| `summary` | str | Short incident title |
| `description` | str | Detailed description |
| `status` | enum | `unknown`, `new`, `in_progress`, `on_hold`, `resolved`, `closed`, `other` |
| `priority` | enum | `unknown`, `low`, `medium`, `high`, `critical`, `other` |
| `severity` | enum | `unknown`, `informational`, `low`, `medium`, `high`, `critical`, `fatal`, `other` |
| `fields` | dict | Custom fields (see below) |
| `tags` | list | Tag names |

### Priority vs Severity

| | Priority | Severity |
|---|----------|----------|
| **Question** | How fast to respond? | How bad is it? |
| **Example** | Critical = drop everything | Fatal = complete system compromise |
| **Set by** | SLA/business impact | Technical impact assessment |

A **high severity** incident (major data breach) might be **medium priority** if contained and no active threat. A **low severity** issue (minor misconfiguration) might be **critical priority** if it affects a VIP client.

## Custom Fields

Custom fields extend the default case schema. Supported types:
`TEXT`, `INTEGER`, `NUMERIC`, `BOOLEAN`, `TIMESTAMP`, `TIMESTAMPTZ`, `JSONB`, `UUID`

## Case Actions (in workflows)

| Action | Description |
|--------|-------------|
| `core.cases.create_case` | Create new case |
| `core.cases.update_case` | Update status/priority/severity |
| `core.cases.get_case` | Get case details |
| `core.cases.list_cases` | List all cases |
| `core.cases.search_cases` | Search by criteria |
| `core.cases.delete_case` | Delete a case |
| `core.cases.create_comment` | Add comment |
| `core.cases.assign_user` | Assign user |
| `core.cases.assign_user_by_email` | Assign by email |
| `core.cases.add_case_tag` | Add tag |
| `core.cases.remove_case_tag` | Remove tag |
| `core.cases.upload_attachment` | Upload file |
| `core.cases.list_attachments` | List attachments |
| `core.cases.download_attachment` | Download attachment |
| `core.cases.list_case_events` | List case events (audit) |

## Pattern: Auto-Case Creation from Alert

```yaml
- ref: create_case
  action: core.cases.create_case
  args:
    summary: "CrowdStrike Alert: ${{ TRIGGER.alert_name }}"
    description: |
      Source: ${{ TRIGGER.source }}
      Hostname: ${{ TRIGGER.hostname }}
      Detection: ${{ TRIGGER.detection_type }}
    priority: ${{ "critical" if TRIGGER.severity == "high" else "medium" }}
    severity: ${{ TRIGGER.severity }}
    tags: ["auto-triage", "crowdstrike"]

- ref: add_enrichment_comment
  action: core.cases.create_comment
  depends_on: [create_case, enrich_ioc]
  args:
    case_id: ${{ ACTIONS.create_case.result.id }}
    content: |
      ## Enrichment Results
      - VT Score: ${{ ACTIONS.enrich_ioc.result.data.data.attributes.last_analysis_stats }}
      - Reputation: ${{ ACTIONS.enrich_ioc.result.data.data.attributes.reputation }}
```

## Pattern: Dual Notification (Audit + Real-time)

```yaml
# Permanent audit trail (case comment)
- ref: audit_comment
  action: core.cases.create_comment
  args:
    case_id: ${{ ACTIONS.create_case.result.id }}
    content: "RTR session established. Batch ID: ${{ ACTIONS.init_rtr.result.batch_id }}"

# Real-time awareness (Slack)
- ref: slack_notify
  action: tools.slack.post_message
  args:
    channel: "#soc-alerts"
    text: "*New Case Created* :rotating_light:\nCase: ${{ ACTIONS.create_case.result.short_id }}\nSummary: ${{ ACTIONS.create_case.result.summary }}"
```

## Pattern: Case Status Update on Resolution

```yaml
- ref: resolve_case
  action: core.cases.update_case
  args:
    case_id: ${{ TRIGGER.case_id }}
    status: resolved
    fields:
      resolution_time: ${{ FN.to_isoformat(FN.now()) }}
      resolved_by: "automated_workflow"

- ref: closing_comment
  action: core.cases.create_comment
  depends_on: [resolve_case]
  args:
    case_id: ${{ TRIGGER.case_id }}
    content: |
      ## Case Resolved
      - Resolution: Automated remediation completed
      - Duration: ${{ FN.hours_between(TRIGGER.created_at, FN.now()) }} hours
      - Actions taken: ${{ FN.serialize_json(TRIGGER.actions_taken) }}
```

## Pattern: Splunk Alert -> Case with Full Context

```yaml
- ref: pull_splunk_context
  action: tools.splunk.search_events
  args:
    query: "search index=main src_ip=${{ TRIGGER.result.src_ip }} | head 20"
    start_time: ${{ FN.to_isoformat(FN.now() - FN.hours(24)) }}
    end_time: ${{ FN.to_isoformat(FN.now()) }}
    limit: 20

- ref: create_case_with_context
  action: core.cases.create_case
  depends_on: [pull_splunk_context]
  args:
    summary: "Splunk Alert: ${{ TRIGGER.search_name }}"
    description: |
      Alert: ${{ TRIGGER.search_name }}
      Source IP: ${{ TRIGGER.result.src_ip }}
      Events found: ${{ FN.length(ACTIONS.pull_splunk_context.result.data.results) }}
    priority: high
    severity: medium
    tags: ["splunk", "auto-triage"]
```

## Best Practices

1. **Always add tags** for filtering: source tool, severity, workflow name
2. **Comments as audit trail**: every significant action gets a case comment
3. **Use `short_id`** in Slack messages for quick reference
4. **Custom fields** for structured data (resolution_time, analyst, MITRE ATT&CK technique)
5. **Search before create**: avoid duplicate cases for the same alert