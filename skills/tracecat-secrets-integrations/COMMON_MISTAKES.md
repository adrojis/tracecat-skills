# Common Mistakes — Secrets & Integrations

## 1. Wrong secret key names
**Wrong:**
```
tracecat_create_secret({
  name: "virustotal",
  keys: [{ key: "TOKEN", value: "xxx" }]
})
```
**Right:**
```
tracecat_create_secret({
  name: "virustotal",
  keys: [{ key: "API_KEY", value: "xxx" }]
})
```
**Why:** Each integration expects specific key names. VirusTotal uses `API_KEY`, not `TOKEN`.

## 2. Wrong secret reference syntax
**Wrong:**
```yaml
api_key: ${{ SECRETS.virustotal }}
api_key: ${{ SECRETS["virustotal"]["API_KEY"] }}
```
**Right:**
```yaml
api_key: ${{ SECRETS.virustotal.API_KEY }}
```
**Why:** Secret access uses dot notation: `SECRETS.<secret_name>.<key_name>`.

## 3. Hardcoding credentials in workflows
**Wrong:**
```yaml
headers:
  x-apikey: abc123def456ghi789
```
**Right:**
```yaml
headers:
  x-apikey: ${{ SECRETS.virustotal.API_KEY }}
```
**Why:** Hardcoded keys are a security risk. Secrets are encrypted, auditable, and rotatable.

## 4. Secret name with spaces or uppercase
**Wrong:**
```
tracecat_create_secret({ name: "Virus Total API" })
tracecat_create_secret({ name: "VIRUSTOTAL" })
```
**Right:**
```
tracecat_create_secret({ name: "virustotal" })
tracecat_create_secret({ name: "crowdstrike" })
```
**Why:** Convention is lowercase, no spaces, matching the provider name.

## 5. Multiple keys for multi-key integrations
**Wrong (separate secrets):**
```
tracecat_create_secret({ name: "crowdstrike_id", keys: [{ key: "VALUE", value: "client_id" }] })
tracecat_create_secret({ name: "crowdstrike_secret", keys: [{ key: "VALUE", value: "client_secret" }] })
```
**Right (single secret, multiple keys):**
```
tracecat_create_secret({
  name: "crowdstrike",
  keys: [
    { key: "CLIENT_ID", value: "xxx" },
    { key: "CLIENT_SECRET", value: "yyy" },
    { key: "BASE_URL", value: "https://api.crowdstrike.com" }
  ]
})
```
**Why:** Related credentials belong in one secret with descriptive key names.

## 6. Using update with wrong identifier
**Wrong:**
```
tracecat_update_secret({ secret_id: "virustotal", keys: [...] })
```
**Right:**
```
tracecat_update_secret({ secret_id: "uuid-from-get-secret", keys: [...] })
```
**Why:** `secret_id` must be the UUID, not the name. Use `tracecat_get_secret` to find the UUID.

## 7. Not testing secret access
**Mistake:** Creating a secret and assuming it works.
**Right:** After creating a secret, run a test workflow that references it:
```yaml
url: https://httpbin.org/headers
method: GET
headers:
  Test-Key: ${{ SECRETS.my_secret.API_KEY }}
```
**Why:** Typos in key names or incorrect values are only caught at runtime.
