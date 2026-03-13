# Common Mistakes — Python Scripts

## 1. Using `code` instead of `script`
**Wrong:**
```yaml
type: core.script.run_python
inputs:
  code: |
    def main():
        return {"ok": True}
```
**Right:**
```yaml
type: core.script.run_python
inputs:
  script: |
    def main():
        return {"ok": True}
```
**Why:** The field name is `script`. Using `code` is silently ignored — the action runs with no script.

## 2. Missing main() function
**Wrong:**
```python
result = [x for x in data if x > 5]
return result
```
**Right:**
```python
def main(data):
    return [x for x in data if x > 5]
```
**Why:** Tracecat expects a `main()` function as the entry point. Top-level code is not executed.

## 3. Returning non-serializable objects
**Wrong:**
```python
def main():
    from datetime import datetime
    return {"time": datetime.now()}
```
**Right:**
```python
def main():
    from datetime import datetime
    return {"time": datetime.now().isoformat()}
```
**Why:** Return values must be JSON-serializable. datetime, set, bytes etc. must be converted.

## 4. Missing allow_network for HTTP calls
**Wrong:**
```yaml
inputs:
  script: |
    import requests
    def main():
        return requests.get("https://api.example.com").json()
  dependencies:
    - requests
```
**Right:**
```yaml
inputs:
  script: |
    import requests
    def main():
        return requests.get("https://api.example.com").json()
  dependencies:
    - requests
  allow_network: true
```
**Why:** Network is disabled by default in the sandbox. Without `allow_network: true`, all HTTP calls fail.

## 5. Not serializing complex inputs
**Wrong:**
```yaml
inputs:
  data: ${{ ACTIONS.fetch.result.items }}
```
**Right:**
```yaml
inputs:
  data: ${{ FN.serialize_json(ACTIONS.fetch.result.items) }}
```
Then in script:
```python
import json
def main(data):
    items = json.loads(data)
    return {"count": len(items)}
```
**Why:** Complex nested objects may not transfer correctly. `FN.serialize_json` + `json.loads` ensures reliable data transfer.

## 6. Missing dependencies
**Wrong:**
```yaml
inputs:
  script: |
    import pandas as pd
    def main(data):
        df = pd.DataFrame(data)
        return df.describe().to_dict()
```
**Right:**
```yaml
inputs:
  script: |
    import pandas as pd
    def main(data):
        df = pd.DataFrame(data)
        return df.describe().to_dict()
  dependencies:
    - pandas
```
**Why:** Non-stdlib packages must be listed in `dependencies` to be pip-installed before execution.

## 7. No error handling for external calls
**Wrong:**
```python
def main(url, api_key):
    import requests
    resp = requests.get(url, headers={"Authorization": f"Bearer {api_key}"})
    return resp.json()
```
**Right:**
```python
def main(url, api_key):
    import requests
    resp = requests.get(url, headers={"Authorization": f"Bearer {api_key}"}, timeout=30)
    resp.raise_for_status()
    return resp.json()
```
**Why:** Without timeout and error handling, scripts can hang or return cryptic errors.

## 8. Printing instead of returning
**Wrong:**
```python
def main(data):
    print(f"Found {len(data)} items")
```
**Right:**
```python
def main(data):
    return {"count": len(data), "message": f"Found {len(data)} items"}
```
**Why:** `print()` output goes to logs but is NOT the action result. Always `return` the result.
