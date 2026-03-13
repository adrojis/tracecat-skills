# Examples — YAML Syntax & Expressions

## Example 1: All Expression Contexts

```yaml
# TRIGGER — webhook/schedule input
src_ip: ${{ TRIGGER.data.src_ip }}
alert_id: ${{ TRIGGER.data.alert.id }}

# ACTIONS — previous action results
score: ${{ ACTIONS.enrich_ip.result.score }}
all_ips: ${{ ACTIONS.get_ips.result }}
error_msg: ${{ ACTIONS.failed_step.error }}

# SECRETS — encrypted credentials
api_key: ${{ SECRETS.virustotal.API_KEY }}
token: ${{ SECRETS.slack.BOT_TOKEN }}

# FN — built-in functions
is_public: ${{ FN.ipv4_is_public(TRIGGER.data.ip) }}
json_str: ${{ FN.serialize_json(ACTIONS.data.result) }}
ip_count: ${{ FN.length(ACTIONS.get_ips.result) }}
upper_name: ${{ FN.upper(TRIGGER.data.hostname) }}

# ENV — environment variables
env_value: ${{ ENV.MY_CUSTOM_VAR }}
```

---

## Example 2: Conditional Expressions (run_if)

```yaml
# Simple truthy
run_if: ${{ ACTIONS.check_ip.result }}

# Negation
run_if: ${{ not ACTIONS.is_whitelisted.result }}

# Comparison
run_if: ${{ ACTIONS.score.result.vt_malicious > 5 }}
run_if: ${{ ACTIONS.scan.result.count == 0 }}
run_if: ${{ ACTIONS.severity.result != "low" }}

# Logical AND
run_if: ${{ ACTIONS.is_malicious.result && FN.ipv4_is_public(ACTIONS.parse.result.ip) }}

# Logical OR
run_if: ${{ ACTIONS.vt_check.result.malicious > 5 || ACTIONS.abuse_check.result.score > 80 }}

# Combined
run_if: ${{ (ACTIONS.is_external.result && ACTIONS.score.result > 70) || ACTIONS.is_known_threat.result }}

# Function result
run_if: ${{ FN.is_not_empty(ACTIONS.fetch.result.items) }}
run_if: ${{ FN.contains(ACTIONS.parse.result.url, "malware") }}
```

---

## Example 3: String Interpolation in Values

```yaml
# URL with expression
url: https://api.virustotal.com/api/v3/ip_addresses/${{ TRIGGER.data.ip }}

# String with multiple expressions
message: "Alert from ${{ TRIGGER.data.source }}: IP ${{ ACTIONS.parse.result.ip }} scored ${{ ACTIONS.score.result.risk }}"

# Multi-line with expressions
content: |
  ## Investigation Report
  - Source IP: ${{ ACTIONS.parse.result.src_ip }}
  - Destination: ${{ ACTIONS.parse.result.dest_ip }}
  - VT Score: ${{ ACTIONS.enrich.result.vt_malicious }}/70
  - Verdict: ${{ ACTIONS.score.result.verdict }}

# Using FN.format for complex formatting
summary: ${{ FN.format("IP {} scored {}/100 ({} detections)", ACTIONS.parse.result.ip, ACTIONS.score.result.composite, ACTIONS.enrich.result.vt_malicious) }}
```

---

## Example 4: Nested Object Access

```yaml
# Deep nested access
country: ${{ ACTIONS.vt_lookup.result.data.attributes.country }}
malicious_count: ${{ ACTIONS.vt_lookup.result.data.attributes.last_analysis_stats.malicious }}

# Array access
first_ip: ${{ ACTIONS.get_ips.result[0] }}
first_detection: ${{ ACTIONS.cs_detections.result.resources[0].detection_id }}

# Conditional value
priority: ${{ FN.conditional(ACTIONS.score.result.risk > 70, "high", "medium") }}
channel: ${{ FN.conditional(ACTIONS.parse.result.severity == "critical", "#critical-alerts", "#alerts") }}
```

---

## Example 5: Complete Action YAML Inputs

```yaml
# HTTP Request — full example
url: https://api.abuseipdb.com/api/v2/check
method: GET
headers:
  Key: ${{ SECRETS.abuseipdb.API_KEY }}
  Accept: application/json
params:
  ipAddress: ${{ ACTIONS.parse_input.result.ip }}
  maxAgeInDays: "90"
  verbose: "true"

# Reshape — aggregating multiple sources
value:
  ip: ${{ ACTIONS.parse.result.ip }}
  enrichment:
    virustotal:
      malicious: ${{ ACTIONS.vt_check.result.data.attributes.last_analysis_stats.malicious }}
      harmless: ${{ ACTIONS.vt_check.result.data.attributes.last_analysis_stats.harmless }}
    abuseipdb:
      score: ${{ ACTIONS.abuse_check.result.data.abuseConfidenceScore }}
      reports: ${{ ACTIONS.abuse_check.result.data.totalReports }}
    greynoise:
      classification: ${{ ACTIONS.gn_check.result.classification }}
      noise: ${{ ACTIONS.gn_check.result.noise }}
  verdict:
    is_malicious: ${{ ACTIONS.vt_check.result.data.attributes.last_analysis_stats.malicious > 5 || ACTIONS.abuse_check.result.data.abuseConfidenceScore > 80 }}
    composite_score: ${{ FN.add(ACTIONS.vt_check.result.data.attributes.last_analysis_stats.malicious, ACTIONS.abuse_check.result.data.abuseConfidenceScore) }}

# Python — with all options
script: |
  import json

  def main(data, threshold):
      items = json.loads(data)
      filtered = [i for i in items if i.get("score", 0) > threshold]
      return {
          "total": len(items),
          "filtered": len(filtered),
          "items": filtered
      }
inputs:
  data: ${{ FN.serialize_json(ACTIONS.fetch.result.items) }}
  threshold: 50
dependencies:
  - none
timeout_seconds: 30
allow_network: false
```

---

## Example 6: Additional FN Functions

```yaml
# --- JSON functions ---
# Deserialize JSON string back to object
parsed: ${{ FN.deserialize_json(ACTIONS.fetch.result.body) }}
# Pretty-print JSON for display
display: ${{ FN.prettify_json(ACTIONS.data.result) }}
# Serialize object to JSON string (for Python inputs)
json_str: ${{ FN.serialize_json(ACTIONS.fetch.result) }}

# --- URL/Encoding functions ---
# Base64url encode (for VirusTotal URL lookups)
encoded_url: ${{ FN.to_base64url(TRIGGER.data.url) }}
# Strip whitespace/padding (combine with to_base64url for VT)
clean: ${{ FN.strip(FN.to_base64url(TRIGGER.data.url)) }}

# --- Time functions ---
# Current timestamp
now: ${{ FN.now() }}
# Time arithmetic (e.g., 60 minutes ago)
one_hour_ago: ${{ FN.now() - FN.minutes(60) }}
# Other time deltas
yesterday: ${{ FN.now() - FN.hours(24) }}
last_week: ${{ FN.now() - FN.days(7) }}

# --- String functions ---
upper: ${{ FN.upper(TRIGGER.data.hostname) }}
lower: ${{ FN.lower(TRIGGER.data.alert_type) }}
contains_check: ${{ FN.contains(ACTIONS.parse.result.url, "malware") }}

# --- Network functions ---
is_public: ${{ FN.ipv4_is_public(TRIGGER.data.ip) }}
# Combine with conditional
action: ${{ FN.conditional(FN.ipv4_is_public(TRIGGER.data.ip), "enrich", "skip") }}

# --- Collection functions ---
length: ${{ FN.length(ACTIONS.get_items.result) }}
is_empty: ${{ FN.is_empty(ACTIONS.fetch.result.items) }}
is_not_empty: ${{ FN.is_not_empty(ACTIONS.fetch.result.items) }}
```
