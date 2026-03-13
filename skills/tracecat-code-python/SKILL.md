---
name: tracecat-code-python
description: Activate when users write Python scripts for core.script.run_python actions in Tracecat workflows
---

# Tracecat Python Code Expert

You are an expert at writing Python scripts for Tracecat workflow actions using `core.script.run_python`.

## Action Configuration

The action type is **`core.script.run_python`** (displayed as "Run Python script" in the UI).

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `script` | string | Yes | — | Python code to execute |
| `inputs` | object | No | — | Key-value pairs passed as function arguments |
| `dependencies` | array | No | — | Pip packages to install at runtime |
| `timeout_seconds` | integer | No | 30 | Max execution time in seconds |
| `allow_network` | boolean | No | false | Enable network access from sandbox |

## Script Rules

1. Script must contain **at least one function**
2. If multiple functions exist, one **must be named `main`**
3. The `main` function's **return value** becomes the action output
4. Return value **must be JSON serializable** (`str`, `int`, `float`, `bool`, `list`, `dict`)

## Accessing Data

### Input data — via function arguments
Inputs are mapped to function parameters **by name**:
```yaml
script: |
  def main(url, threshold):
      if threshold > 50:
          return {"status": "high", "target": url}
      return {"status": "low", "target": url}

inputs:
  url: ${{ TRIGGER.data.target_url }}
  threshold: ${{ ACTIONS.score_risk.result.score }}
```

- Extra input keys not in the function signature are **silently ignored**
- Missing parameters receive `None`
- Default parameter values are supported

### Secrets — passed via inputs using expressions
You **cannot** access secrets directly from Python code. Pass them through `inputs`:
```yaml
inputs:
  api_key: ${{ SECRETS.virustotal.API_KEY }}
```

### Trigger and action data — passed via inputs
```yaml
inputs:
  alert_data: ${{ TRIGGER.data }}
  enrichment: ${{ ACTIONS.enrich_ip.result }}
```

## Available Modules

### Standard library (always available)
`json`, `re`, `datetime`, `math`, `collections`, `itertools`, `hashlib`, `base64`, `urllib.parse`, `ipaddress`, `csv`, `io`, `os.path`, `uuid`, `typing`, and all other Python 3.12 stdlib modules.

### Third-party packages (via `dependencies`)
Any pip-installable package. Must set `allow_network: true`:
```yaml
dependencies:
  - requests
  - numpy
allow_network: true
```
Packages are installed at runtime via `uv` package manager.

## Common Patterns

### Pattern 1: Data transformation
```yaml
script: |
  def main(items):
      return [
          {"name": item["name"].upper(), "score": item["value"] * 10}
          for item in items
          if item["value"] > 0
      ]

inputs:
  items: ${{ ACTIONS.fetch_data.result.records }}
```

### Pattern 2: IP/IOC parsing
```yaml
script: |
  import re
  import ipaddress

  def main(raw_text):
      ips = re.findall(r'\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b', raw_text)
      valid = []
      for ip in ips:
          try:
              addr = ipaddress.ip_address(ip)
              if not addr.is_private:
                  valid.append(str(addr))
          except ValueError:
              continue
      return {"public_ips": list(set(valid)), "count": len(valid)}

inputs:
  raw_text: ${{ TRIGGER.data.log_entry }}
```

### Pattern 3: API call with external library
```yaml
script: |
  import requests

  def main(domain, api_key):
      resp = requests.get(
          f"https://api.example.com/lookup/{domain}",
          headers={"Authorization": f"Bearer {api_key}"},
          timeout=10
      )
      resp.raise_for_status()
      return resp.json()

inputs:
  domain: ${{ TRIGGER.data.domain }}
  api_key: ${{ SECRETS.example_api.API_KEY }}
dependencies:
  - requests
allow_network: true
timeout_seconds: 15
```

### Pattern 4: Aggregation / statistics
```yaml
script: |
  def main(events):
      by_type = {}
      for event in events:
          t = event.get("type", "unknown")
          by_type[t] = by_type.get(t, 0) + 1

      total = len(events)
      return {
          "total": total,
          "by_type": by_type,
          "top_type": max(by_type, key=by_type.get) if by_type else None
      }

inputs:
  events: ${{ ACTIONS.query_siem.result.hits }}
```

### Pattern 5: Hash computation
```yaml
script: |
  import hashlib

  def main(content):
      return {
          "md5": hashlib.md5(content.encode()).hexdigest(),
          "sha256": hashlib.sha256(content.encode()).hexdigest()
      }

inputs:
  content: ${{ TRIGGER.data.file_content }}
```

### Pattern 6: Multiple helper functions
```yaml
script: |
  import json
  from datetime import datetime, timezone

  def parse_timestamp(ts):
      return datetime.fromisoformat(ts).replace(tzinfo=timezone.utc)

  def is_recent(ts, hours=24):
      delta = datetime.now(timezone.utc) - parse_timestamp(ts)
      return delta.total_seconds() < hours * 3600

  def main(alerts):
      recent = [a for a in alerts if is_recent(a["timestamp"])]
      return {
          "total": len(alerts),
          "recent_24h": len(recent),
          "recent_alerts": recent
      }

inputs:
  alerts: ${{ ACTIONS.fetch_alerts.result.data }}
```

## Sandbox Environment

| Aspect | Detail |
|---|---|
| Python version | 3.12 (slim-bookworm) |
| Package manager | uv 0.9.15 |
| User | `sandbox` (UID 1000, non-root) |
| Default timeout | 30 seconds |
| Network | Disabled by default |
| Isolation | nsjail on Kubernetes, subprocess on Docker Compose |

## Limitations

1. **No direct secret access** — always pass via `inputs` with `${{ SECRETS.* }}`
2. **Error traces suppressed** — detailed Python errors hidden for security (issue #1347). Debug by simplifying scripts and isolating the error.
3. **Network off by default** — must explicitly set `allow_network: true` for HTTP calls or pip dependencies
4. **JSON serializable output only** — no datetime, bytes, or custom objects in return value
5. **No persistent state** — each execution is isolated, no shared filesystem between runs
6. **Dependencies installed each run** — no caching between executions, adds latency

## Tips

- Keep scripts focused on one task — prefer multiple simple actions over one complex script
- Always add `timeout_seconds` for scripts making HTTP calls
- Use `allow_network: true` only when needed (dependencies or outbound calls)
- Test complex logic locally before deploying
- For reusable logic across workflows, consider UDFs (User-Defined Functions) via the registry instead

## Related Skills
- **tracecat-mcp-tools-expert** — MCP tool reference for action creation/updates
- **tracecat-workflow-patterns** — Workflow design patterns using Python actions
- **tracecat-yaml-syntax** — YAML syntax for script inputs
- **tracecat-validation-debug** — Debug Python script execution errors
- **tracecat-integration-expert** — Custom API integrations via Python

## Reference Files
- [Common Mistakes](./COMMON_MISTAKES.md)
- [Examples](./examples.md)
- [README](./README.md)
