# Common Mistakes — Integrations

## 1. Using http_request when native integration exists
**Wrong:**
```yaml
type: core.http_request
inputs:
  url: https://www.virustotal.com/api/v3/files/${{ TRIGGER.hash }}
  method: GET
  headers:
    x-apikey: ${{ SECRETS.virustotal.API_KEY }}
```
**Right:**
```yaml
type: tools.virustotal.get_file_report
inputs:
  hash: ${{ TRIGGER.hash }}
```
**Why:** Native integrations handle authentication, error handling, and response parsing automatically.

## 2. Wrong secret key names
**Wrong:**
```yaml
headers:
  Authorization: ${{ SECRETS.virustotal.TOKEN }}
```
**Right:**
```yaml
headers:
  x-apikey: ${{ SECRETS.virustotal.API_KEY }}
```
**Why:** Each integration expects specific key names. Check the integration docs or tracecat-secrets-integrations skill.

## 3. Missing authentication header
**Wrong:**
```yaml
url: https://api.abuseipdb.com/api/v2/check
method: GET
params:
  ipAddress: ${{ TRIGGER.ip }}
```
**Right:**
```yaml
url: https://api.abuseipdb.com/api/v2/check
method: GET
headers:
  Key: ${{ SECRETS.abuseipdb.API_KEY }}
  Accept: application/json
params:
  ipAddress: ${{ TRIGGER.ip }}
```
**Why:** Most APIs require an authentication header. Check the specific API documentation.

## 4. Hardcoded API keys
**Wrong:**
```yaml
headers:
  x-apikey: abc123def456
```
**Right:**
```yaml
headers:
  x-apikey: ${{ SECRETS.virustotal.API_KEY }}
```
**Why:** Never hardcode credentials. Use Tracecat secrets for secure storage and rotation.

## 5. Not handling API rate limits
**Wrong:**
```yaml
# No retry policy, no delay
type: core.http_request
inputs:
  url: https://api.virustotal.com/api/v3/ip_addresses/${{ ACTIONS.get_ips.result }}
control_flow:
  for_each: ${{ ACTIONS.get_ips.result }}
```
**Right:**
```yaml
type: core.http_request
inputs:
  url: https://api.virustotal.com/api/v3/ip_addresses/${{ ACTIONS.get_ips.result }}
control_flow:
  for_each: ${{ ACTIONS.get_ips.result }}
  start_delay: 2
  retry_policy:
    max_attempts: 3
    timeout: 30
```
**Why:** APIs have rate limits. Add delays between iterations and retries for transient failures.

## 6. Wrong Content-Type for POST requests
**Wrong:**
```yaml
method: POST
headers:
  Authorization: Bearer ${{ SECRETS.api.TOKEN }}
payload:
  data: "some value"
```
**Right:**
```yaml
method: POST
headers:
  Authorization: Bearer ${{ SECRETS.api.TOKEN }}
  Content-Type: application/json
payload:
  data: "some value"
```
**Why:** Tracecat's http_request defaults may not set Content-Type. Explicit is safer for APIs that require it.

## 7. Ignoring API error responses
**Wrong:** No error path after http_request.
**Right:** Add an error edge from the HTTP action to an error handler:
```yaml
# In graph: add edge with source_handle: "error" to error_handler action
```
**Why:** External API calls can fail (rate limits, auth expired, service down). Always plan for failure.

## 8. VirusTotal URL lookup without proper base64url encoding
**Wrong:**
```yaml
url: https://www.virustotal.com/api/v3/urls/${{ TRIGGER.data.url }}
```
**Right:**
```yaml
url: https://www.virustotal.com/api/v3/urls/${{ FN.strip(FN.to_base64url(TRIGGER.data.url)) }}
```
**Why:** VirusTotal URL endpoint requires the URL to be base64url-encoded (RFC 4648). `FN.to_base64url` encodes it, and `FN.strip` removes trailing `=` padding that VT doesn't accept.

## 9. Missing verify_ssl for self-signed APIs
**Wrong:**
```yaml
type: core.http_request
inputs:
  url: https://wazuh.internal:55000/agents
  method: GET
  headers:
    Authorization: Bearer ${{ SECRETS.wazuh.API_KEY }}
```
**Right:**
```yaml
type: core.http_request
inputs:
  url: ${{ SECRETS.wazuh.SERVER_URL }}/agents
  method: GET
  headers:
    Authorization: Bearer ${{ SECRETS.wazuh.API_KEY }}
  verify_ssl: false
```
**Why:** On-premise tools (Wazuh, MISP, TheHive) often use self-signed certificates. Without `verify_ssl: false`, the request fails with SSL verification error.
