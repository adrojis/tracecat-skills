# Examples — Action Configuration

## Example 1: HTTP Request to VirusTotal

```yaml
type: core.http_request
title: Check IP on VirusTotal
inputs:
  url: https://www.virustotal.com/api/v3/ip_addresses/${{ TRIGGER.data.src_ip }}
  method: GET
  headers:
    x-apikey: ${{ SECRETS.virustotal.API_KEY }}
    Accept: application/json
```

**MCP command:**
```
tracecat_update_action({
  action_id: "act_xxx",
  workflow_id: "wf_xxx",
  inputs: "url: https://www.virustotal.com/api/v3/ip_addresses/${{ TRIGGER.data.src_ip }}\nmethod: GET\nheaders:\n  x-apikey: ${{ SECRETS.virustotal.API_KEY }}\n  Accept: application/json"
})
```

---

## Example 2: Reshape with Conditional Logic

```yaml
type: core.transform.reshape
title: Build Risk Assessment
inputs:
  value:
    ip: ${{ TRIGGER.data.src_ip }}
    vt_malicious: ${{ ACTIONS.vt_check.result.data.attributes.last_analysis_stats.malicious }}
    abuse_score: ${{ ACTIONS.abuse_check.result.data.abuseConfidenceScore }}
    is_high_risk: ${{ ACTIONS.vt_check.result.data.attributes.last_analysis_stats.malicious > 5 }}
    priority: ${{ FN.conditional(ACTIONS.vt_check.result.data.attributes.last_analysis_stats.malicious > 5, "high", "medium") }}
    summary: ${{ FN.format("VT:{}, Abuse:{}", ACTIONS.vt_check.result.data.attributes.last_analysis_stats.malicious, ACTIONS.abuse_check.result.data.abuseConfidenceScore) }}
```

---

## Example 3: Python Script with Dependencies

```yaml
type: core.script.run_python
title: Deduplicate and Score IOCs
inputs:
  script: |
    import json

    def main(raw_iocs, threshold):
        iocs = json.loads(raw_iocs)
        seen = set()
        unique = []
        for ioc in iocs:
            key = ioc.get("value", "")
            if key not in seen:
                seen.add(key)
                unique.append(ioc)
        high_risk = [i for i in unique if i.get("score", 0) > threshold]
        return {
            "total": len(iocs),
            "unique": len(unique),
            "high_risk_count": len(high_risk),
            "high_risk_iocs": high_risk
        }
  inputs:
    raw_iocs: ${{ FN.serialize_json(ACTIONS.fetch_iocs.result) }}
    threshold: 70
  dependencies: []
  timeout_seconds: 30
```

---

## Example 4: Case Creation with Full Context

```yaml
type: core.cases.create_case
title: Create Investigation Case
inputs:
  case_title: "[ALERT] Malicious activity from ${{ ACTIONS.parse_alert.result.src_ip }}"
  payload:
    alert_source: ${{ TRIGGER.data.source }}
    src_ip: ${{ ACTIONS.parse_alert.result.src_ip }}
    dest_ip: ${{ ACTIONS.parse_alert.result.dest_ip }}
    vt_score: ${{ ACTIONS.enrich.result.vt_malicious }}
    abuse_score: ${{ ACTIONS.enrich.result.abuse_score }}
    first_seen: ${{ TRIGGER.data.timestamp }}
  status: new
  priority: ${{ ACTIONS.score.result.priority }}
  malice: ${{ FN.conditional(ACTIONS.score.result.is_malicious, "malicious", "unknown") }}
  action: "Investigate source IP and block if confirmed malicious"
```

---

## Example 5: Conditional Execution with Retry

```yaml
type: core.http_request
title: Block IP on Firewall
control_flow:
  run_if: ${{ ACTIONS.score.result.is_malicious && FN.ipv4_is_public(ACTIONS.parse_alert.result.src_ip) }}
  retry_policy:
    max_attempts: 3
    timeout: 30
inputs:
  url: ${{ SECRETS.firewall.API_URL }}/api/block
  method: POST
  headers:
    Authorization: Bearer ${{ SECRETS.firewall.API_KEY }}
    Content-Type: application/json
  payload:
    ip: ${{ ACTIONS.parse_alert.result.src_ip }}
    duration: "24h"
    reason: "Automated containment - VT score ${{ ACTIONS.enrich.result.vt_malicious }}"
```
