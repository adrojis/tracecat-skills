# Tracecat Python Script Examples

## Example 1: Deduplication Filter

Remove duplicate IOCs from a list before enrichment:

```yaml
action: core.script.run_python
args:
  script: |
    def main(iocs):
        seen = set()
        unique = []
        for ioc in iocs:
            key = (ioc["type"], ioc["value"])
            if key not in seen:
                seen.add(key)
                unique.append(ioc)
        return {"unique_iocs": unique, "duplicates_removed": len(iocs) - len(unique)}

  inputs:
    iocs: ${{ ACTIONS.extract_iocs.result.indicators }}
```

## Example 2: CVSS Score Classifier

Classify vulnerabilities by CVSS score:

```yaml
action: core.script.run_python
args:
  script: |
    def classify(score):
        if score >= 9.0:
            return "critical"
        elif score >= 7.0:
            return "high"
        elif score >= 4.0:
            return "medium"
        return "low"

    def main(vulnerabilities):
        classified = []
        for vuln in vulnerabilities:
            vuln["severity"] = classify(vuln.get("cvss", 0))
            classified.append(vuln)

        by_severity = {}
        for v in classified:
            s = v["severity"]
            by_severity[s] = by_severity.get(s, 0) + 1

        return {
            "vulnerabilities": classified,
            "summary": by_severity
        }

  inputs:
    vulnerabilities: ${{ ACTIONS.scan_results.result.findings }}
```

## Example 3: Email Header Parser

Parse email headers for phishing analysis:

```yaml
action: core.script.run_python
args:
  script: |
    import re

    def extract_ips(header_value):
        return re.findall(r'\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b', header_value)

    def main(raw_headers):
        results = {
            "from": None,
            "reply_to": None,
            "received_ips": [],
            "spf": None,
            "dkim": None,
            "suspicious": []
        }

        for header in raw_headers:
            name = header["name"].lower()
            value = header["value"]

            if name == "from":
                results["from"] = value
            elif name == "reply-to":
                results["reply_to"] = value
            elif name == "received":
                results["received_ips"].extend(extract_ips(value))
            elif name == "received-spf":
                results["spf"] = "pass" if "pass" in value.lower() else "fail"
            elif name == "dkim-signature":
                results["dkim"] = "present"

        # Suspicious indicators
        if results["from"] and results["reply_to"]:
            from_domain = results["from"].split("@")[-1].rstrip(">")
            reply_domain = results["reply_to"].split("@")[-1].rstrip(">")
            if from_domain != reply_domain:
                results["suspicious"].append("from/reply-to domain mismatch")

        if results["spf"] == "fail":
            results["suspicious"].append("SPF check failed")

        return results

  inputs:
    raw_headers: ${{ TRIGGER.data.email_headers }}
```

## Example 4: Log Correlation

Correlate events from multiple sources by timestamp window:

```yaml
action: core.script.run_python
args:
  script: |
    from datetime import datetime

    def parse_ts(ts_str):
        for fmt in ["%Y-%m-%dT%H:%M:%S.%fZ", "%Y-%m-%dT%H:%M:%SZ", "%Y-%m-%d %H:%M:%S"]:
            try:
                return datetime.strptime(ts_str, fmt)
            except ValueError:
                continue
        return None

    def main(auth_events, network_events, window_seconds):
        window = window_seconds or 300
        correlated = []

        for auth in auth_events:
            auth_ts = parse_ts(auth.get("timestamp", ""))
            if not auth_ts:
                continue

            related_network = []
            for net in network_events:
                net_ts = parse_ts(net.get("timestamp", ""))
                if not net_ts:
                    continue
                if abs((auth_ts - net_ts).total_seconds()) <= window:
                    if auth.get("source_ip") == net.get("source_ip"):
                        related_network.append(net)

            if related_network:
                correlated.append({
                    "auth_event": auth,
                    "related_network": related_network,
                    "correlation_count": len(related_network)
                })

        return {
            "correlated_events": correlated,
            "total_correlations": len(correlated)
        }

  inputs:
    auth_events: ${{ ACTIONS.query_auth_logs.result.events }}
    network_events: ${{ ACTIONS.query_network_logs.result.events }}
    window_seconds: 300
```

## Example 5: External API with Retry Logic

Call an external API with manual retry handling:

```yaml
action: core.script.run_python
args:
  script: |
    import requests
    import time

    def main(url, api_key, max_retries):
        retries = max_retries or 3
        last_error = None

        for attempt in range(retries):
            try:
                resp = requests.get(
                    url,
                    headers={"Authorization": f"Bearer {api_key}"},
                    timeout=10
                )
                if resp.status_code == 429:
                    time.sleep(2 ** attempt)
                    continue
                resp.raise_for_status()
                return {"data": resp.json(), "attempts": attempt + 1}
            except Exception as e:
                last_error = str(e)
                time.sleep(1)

        return {"error": last_error, "attempts": retries, "data": None}

  inputs:
    url: ${{ ACTIONS.build_url.result.endpoint }}
    api_key: ${{ SECRETS.threat_intel.API_KEY }}
    max_retries: 3
  dependencies:
    - requests
  allow_network: true
  timeout_seconds: 45
```

## Example 6: JSON Report Generator

Build a structured report from multiple enrichment sources:

```yaml
action: core.script.run_python
args:
  script: |
    from datetime import datetime, timezone
    import json

    def main(alert, vt_result, abuse_result, whois_result):
        report = {
            "generated_at": datetime.now(timezone.utc).isoformat(),
            "alert_summary": {
                "type": alert.get("type"),
                "severity": alert.get("severity"),
                "source": alert.get("source_ip")
            },
            "enrichment": {
                "virustotal": {
                    "malicious": vt_result.get("malicious", 0),
                    "suspicious": vt_result.get("suspicious", 0),
                    "verdict": "malicious" if vt_result.get("malicious", 0) > 0 else "clean"
                },
                "abuseipdb": {
                    "confidence_score": abuse_result.get("abuse_confidence_score", 0),
                    "total_reports": abuse_result.get("total_reports", 0),
                    "is_abusive": abuse_result.get("abuse_confidence_score", 0) > 50
                },
                "whois": {
                    "registrar": whois_result.get("registrar"),
                    "creation_date": whois_result.get("creation_date"),
                    "country": whois_result.get("country")
                }
            },
            "risk_assessment": {
                "score": min(100, (
                    vt_result.get("malicious", 0) * 20 +
                    abuse_result.get("abuse_confidence_score", 0)
                ) // 2),
                "recommendation": ""
            }
        }

        score = report["risk_assessment"]["score"]
        if score >= 70:
            report["risk_assessment"]["recommendation"] = "Block immediately and investigate"
        elif score >= 40:
            report["risk_assessment"]["recommendation"] = "Monitor closely and gather more context"
        else:
            report["risk_assessment"]["recommendation"] = "Low risk — log and continue monitoring"

        return report

  inputs:
    alert: ${{ TRIGGER.data }}
    vt_result: ${{ ACTIONS.check_virustotal.result }}
    abuse_result: ${{ ACTIONS.check_abuseipdb.result }}
    whois_result: ${{ ACTIONS.check_whois.result }}
```
