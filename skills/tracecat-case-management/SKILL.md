---
name: tracecat-case-management
description: Activate when users manage security cases, track incidents, or configure case lifecycle workflows in Tracecat
---

# Tracecat Case Management

You are an expert at managing security cases in Tracecat. Use this guide for best practices in incident tracking and case lifecycle management.

## Case Lifecycle

```
new → in_progress → resolved → closed
                  → escalated → in_progress → resolved → closed
```

## Case Fields

| Field | Values | Description |
|-------|--------|-------------|
| `status` | `new`, `in_progress`, `resolved`, `closed` | Current state |
| `priority` | `low`, `medium`, `high`, `critical` | Urgency level |
| `malice` | `benign`, `malicious`, `unknown` | Threat determination |
| `action` | Free text | Recommended or taken action |
| `payload` | JSON object | Alert/event data |
| `tags` | JSON object | Custom categorization |

## Priority Matrix

| Severity | Business Impact | Priority |
|----------|----------------|----------|
| Critical vuln + Active exploit | Data breach risk | **critical** |
| High severity alert | Service degradation | **high** |
| Medium finding | Potential risk | **medium** |
| Informational | No immediate risk | **low** |

## Best Practices

### Creating Cases
- Always link to the source workflow via `workflow_id`
- Include the raw alert/event data in `payload`
- Set initial `malice` to `unknown` until investigation completes
- Use descriptive titles: `"[ALERT] Suspicious login from {country} for {user}"`

### During Investigation
1. Set status to `in_progress` immediately when starting
2. Add comments for every significant finding
3. Update `malice` once determination is made
4. Update `priority` if risk assessment changes

### Closing Cases
1. Add a summary comment with:
   - Root cause (if malicious)
   - Actions taken
   - Recommendations
2. Set `malice` to final determination
3. Set `action` to describe resolution
4. Set status to `resolved`, then `closed`

### Comment Templates

**Triage comment:**
```
## Initial Triage
- **Source:** {alert_source}
- **Affected:** {affected_assets}
- **Initial Assessment:** {assessment}
- **Next Steps:** {planned_actions}
```

**Resolution comment:**
```
## Resolution
- **Root Cause:** {root_cause}
- **Actions Taken:** {actions}
- **Impact:** {impact_assessment}
- **Recommendations:** {recommendations}
```

## MCP Tools for Case Management

```
# List open cases
tracecat_list_cases(status="new")
tracecat_list_cases(status="in_progress")

# Create a case
tracecat_create_case(
  workflow_id="wf:...",
  case_title="[ALERT] Suspicious activity detected",
  priority="high",
  malice="unknown"
)

# Update during investigation
tracecat_update_case(case_id="...", status="in_progress")
tracecat_add_comment(case_id="...", content="## Triage\n...")

# Close case
tracecat_update_case(case_id="...", status="resolved", malice="malicious")
tracecat_add_comment(case_id="...", content="## Resolution\n...")
```

## Related Skills
- **tracecat-mcp-tools-expert** — MCP tool reference for case operations
- **tracecat-workflow-patterns** — Alert triage and incident response patterns
- **tracecat-validation-debug** — Debug case creation failures
- **tracecat-code-python** — Custom case enrichment scripts

## Reference Files
- [Common Mistakes](./COMMON_MISTAKES.md)
- [Examples](./EXAMPLES.md)
- [README](./README.md)
