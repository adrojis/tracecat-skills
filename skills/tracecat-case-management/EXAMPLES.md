# Examples — Case Management

## Example 1: SOC Alert Triage Case

**Create case from alert triage workflow:**
```
tracecat_create_case({
  workflow_id: "wf_triage",
  case_title: "[PHISHING] Suspicious email from external sender",
  payload: {
    alert_id: "ALT-2024-1234",
    sender: "attacker@evil.com",
    subject: "Urgent: Update your credentials",
    recipients: ["user1@company.com", "user2@company.com"],
    urls_found: ["https://evil.com/phish"],
    vt_score: 12,
    detection_rule: "phishing_url_in_body"
  },
  status: "new",
  priority: "high",
  malice: "malicious",
  action: "Block sender, quarantine emails, notify affected users"
})
```

---

## Example 2: Case Lifecycle — Investigation Flow

```
# 1. Create case
tracecat_create_case({
  workflow_id: "wf_xxx",
  case_title: "[MALWARE] Suspicious EXE on SharePoint",
  payload: { file_hash: "abc123", uploader: "john.doe" },
  status: "new",
  priority: "critical",
  malice: "unknown"
})

# 2. Start investigation — add comment
tracecat_add_comment({
  case_id: "case_xxx",
  content: "## Investigation Started\n- Analyst: SOC-L2\n- File submitted to VT and GLIMPS for analysis\n- Checking uploader activity in Azure AD"
})

# 3. Update status
tracecat_update_case({
  case_id: "case_xxx",
  status: "in_progress",
  action: "Awaiting sandbox analysis results"
})

# 4. Add findings
tracecat_add_comment({
  case_id: "case_xxx",
  content: "## Analysis Results\n- VT: 45/70 detections (malicious)\n- GLIMPS: Trojan.GenericKD\n- Uploader account shows unusual login from VPN\n\n**Recommendation:** Block file hash, disable uploader account"
})

# 5. Resolve
tracecat_update_case({
  case_id: "case_xxx",
  status: "resolved",
  malice: "malicious",
  action: "File blocked, account disabled, incident documented"
})
```

---

## Example 3: Workflow Action — Auto-Create Case

```yaml
# In a workflow: create case when VT score is high
type: core.cases.create_case
title: Auto-Create Case
control_flow:
  run_if: ${{ ACTIONS.score_risk.result.is_high_risk }}
inputs:
  case_title: "[AUTO] High-risk IP detected: ${{ ACTIONS.parse.result.src_ip }}"
  payload:
    src_ip: ${{ ACTIONS.parse.result.src_ip }}
    vt_score: ${{ ACTIONS.vt_check.result.data.attributes.last_analysis_stats.malicious }}
    abuse_score: ${{ ACTIONS.abuse_check.result.data.abuseConfidenceScore }}
    detection_time: ${{ TRIGGER.data.timestamp }}
  status: new
  priority: high
  malice: malicious
  action: "Block IP and investigate lateral movement"
```

---

## Example 4: Case Comment Templates

**Triage comment:**
```yaml
type: core.cases.create_comment
title: Add Triage Summary
inputs:
  case_id: ${{ ACTIONS.create_case.result.id }}
  content: |
    ## Triage Summary
    - **Source:** ${{ TRIGGER.data.source }}
    - **Detection Rule:** ${{ TRIGGER.data.rule_name }}
    - **VT Score:** ${{ ACTIONS.enrich.result.vt_malicious }}/70
    - **AbuseIPDB:** ${{ ACTIONS.enrich.result.abuse_score }}%
    - **Public IP:** ${{ FN.ipv4_is_public(TRIGGER.data.src_ip) }}

    ### Recommendation
    ${{ FN.conditional(ACTIONS.score.result.is_high_risk, "Immediate containment required", "Monitor and investigate") }}
```

---

## Example 5: Bulk Case Query and Update

```
# List all new critical cases
tracecat_list_cases({ status: "new", limit: 50 })

# Update multiple cases to in_progress
tracecat_update_case({ case_id: "case_1", status: "in_progress" })
tracecat_update_case({ case_id: "case_2", status: "in_progress" })

# Close resolved cases
tracecat_update_case({
  case_id: "case_old",
  status: "closed",
  action: "No further action — false positive confirmed"
})
```
