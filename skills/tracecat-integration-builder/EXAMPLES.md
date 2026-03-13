# Examples — Integration Builder

## Example 1: "Intègre-moi VirusTotal" — URL donnée par l'utilisateur

**User:** "Intègre VirusTotal dans Tracecat, voici la doc : https://docs.virustotal.com/reference/overview"

### Step 1: Research
- WebFetch la doc → REST API, API key auth via header `x-apikey`
- Endpoints SOAR-utiles : `/files/{id}`, `/urls`, `/ip_addresses/{ip}`, `/domains/{domain}`
- Rate limit : 4 req/min (free), 500 req/min (premium)
- OpenAPI spec dispo : `https://www.virustotal.com/api/v3/openapi.json`

### Step 2: Strategy
OpenAPI spec existe → **Path A** (converter) possible, mais les 4-5 endpoints clés sont simples → **Path B** (manual) est aussi rapide et plus contrôlé.

### Step 3: Build (Path B — manual)

```
# 1. Create workflow
tracecat_create_workflow:
  title: "VirusTotal Enrichment"
  description: "Enrich IOCs via VirusTotal API"

# 2. Ask user for API key, then create secret
tracecat_create_secret:
  name: "virustotal"
  keys:
    - key: "API_KEY"
      value: "<user-provided>"

# 3. Create actions
tracecat_create_action:
  workflow_id: "wf_xxx"
  type: "core.http_request"
  title: "Check IP on VirusTotal"

# 4. Configure inputs
tracecat_update_action:
  action_id: "act_xxx"
  workflow_id: "wf_xxx"
  inputs: |
    url: https://www.virustotal.com/api/v3/ip_addresses/${{ TRIGGER.data.ip }}
    method: GET
    headers:
      x-apikey: ${{ SECRETS.virustotal.API_KEY }}
      Accept: application/json

# 5. Connect edges + position nodes
# 6. Validate + deploy
```

---

## Example 2: "Intègre Wazuh" — Service self-hosted, pas d'OpenAPI spec

**User:** "Voici notre Wazuh : https://wazuh.com/platform/api/"

### Step 1: Research
- WebFetch → REST API, auth = Basic → JWT token
- Pas d'OpenAPI spec officiel
- Endpoints clés : `/security/user/authenticate`, `/alerts`, `/agents`, `/syscheck`
- Self-hosted → verify_ssl: false probable

### Step 2: Strategy
Pas de spec → **Path B** (manual build) obligatoire

### Step 3: Build

```
# 1. Create workflow
tracecat_create_workflow:
  title: "Wazuh Alert Monitoring"

# 2. Secret (demander user/password à l'utilisateur)
tracecat_create_secret:
  name: "wazuh"
  keys:
    - key: "API_USER"
      value: "wazuh-wui"
    - key: "API_PASSWORD"
      value: "<user-provided>"
    - key: "BASE_URL"
      value: "https://wazuh-manager:55000"

# 3. Action 1: Auth (get JWT)
tracecat_create_action:
  type: "core.http_request"
  title: "Wazuh Auth"

tracecat_update_action:
  inputs: |
    url: ${{ SECRETS.wazuh.BASE_URL }}/security/user/authenticate
    method: POST
    headers:
      Authorization: Basic ${{ FN.to_base64(SECRETS.wazuh.API_USER + ':' + SECRETS.wazuh.API_PASSWORD) }}
    verify_ssl: false

# 4. Action 2: Get Alerts (uses JWT from auth)
tracecat_create_action:
  type: "core.http_request"
  title: "Get Wazuh Alerts"

tracecat_update_action:
  inputs: |
    url: ${{ SECRETS.wazuh.BASE_URL }}/alerts
    method: GET
    headers:
      Authorization: Bearer ${{ ACTIONS.wazuh_auth.result.data.token }}
    params:
      limit: 50
      sort: -timestamp
    verify_ssl: false

# 5. Edges: trigger → auth → get_alerts
# 6. Position, validate, deploy
```

---

## Example 3: "Intègre Splunk" — Webhook push + query

**User:** "On utilise Splunk, voici la doc : https://docs.splunk.com/Documentation/Splunk/latest/RESTREF/RESTprolog"

### Step 1: Research
- Deux modes : Splunk POUSSE des alertes (webhook) ET on QUERY Splunk (REST API)
- Auth REST : Bearer token (session key) ou Basic auth
- Pas d'OpenAPI spec officiel

### Step 2: Strategy
Deux intégrations à combiner :
- **Path C** : Webhook trigger pour recevoir les alertes Splunk
- **Path B** : Actions manuelles pour query Splunk

### Step 3: Build

**Workflow 1 — Recevoir les alertes Splunk (Path C)**
```
# Créer workflow avec webhook trigger
tracecat_create_workflow:
  title: "Splunk Alert Receiver"

# Activer le webhook
# Donner l'URL webhook à l'utilisateur pour la configurer dans Splunk
# → Actions downstream : extract IOCs, enrich, create case
```

**Workflow 2 — Query Splunk (Path B)**
```
# Secret
tracecat_create_secret:
  name: "splunk"
  keys:
    - key: "TOKEN"
      value: "<user-provided>"
    - key: "BASE_URL"
      value: "https://splunk:8089"

# Action: Search
tracecat_update_action:
  inputs: |
    url: ${{ SECRETS.splunk.BASE_URL }}/services/search/jobs
    method: POST
    headers:
      Authorization: Bearer ${{ SECRETS.splunk.TOKEN }}
    payload:
      search: "search index=main sourcetype=syslog error | head 100"
      output_mode: json
    verify_ssl: false
```

---

## Example 4: "Intègre CrowdStrike" — OAuth2

**User:** "Voici CrowdStrike : https://falcon.crowdstrike.com/documentation/page/a2a7fc0e/crowdstrike-oauth2-based-apis"

### Step 1: Research
- OAuth2 client credentials → access_token (expire 30min)
- OpenAPI spec dispo (Swagger 2.0) → convertible en 3.0
- Endpoints clés : detects, devices, incidents, indicators

### Step 2: Strategy
Spec existe mais OAuth2 complexe → **Path B** avec auth en première action du workflow

### Step 3: Build

```
# Secret
tracecat_create_secret:
  name: "crowdstrike"
  keys:
    - key: "CLIENT_ID"
      value: "<user-provided>"
    - key: "CLIENT_SECRET"
      value: "<user-provided>"
    - key: "BASE_URL"
      value: "https://api.crowdstrike.com"

# Action 1: Get OAuth2 Token
tracecat_update_action:
  inputs: |
    url: ${{ SECRETS.crowdstrike.BASE_URL }}/oauth2/token
    method: POST
    headers:
      Content-Type: application/x-www-form-urlencoded
    payload:
      client_id: ${{ SECRETS.crowdstrike.CLIENT_ID }}
      client_secret: ${{ SECRETS.crowdstrike.CLIENT_SECRET }}

# Action 2: Search Detections (uses token from action 1)
tracecat_update_action:
  inputs: |
    url: ${{ SECRETS.crowdstrike.BASE_URL }}/detects/queries/detects/v1
    method: GET
    headers:
      Authorization: Bearer ${{ ACTIONS.get_token.result.access_token }}
    params:
      filter: "status:'new'"
      limit: 50

# Action 3: Get Detection Details
tracecat_update_action:
  inputs: |
    url: ${{ SECRETS.crowdstrike.BASE_URL }}/detects/entities/summaries/GET/v1
    method: POST
    headers:
      Authorization: Bearer ${{ ACTIONS.get_token.result.access_token }}
    payload:
      ids: ${{ ACTIONS.search_detections.result.resources }}

# Edges: trigger → get_token → search → get_details → create_case
```

---

## Example 5: "Intègre TheHive" — Service communautaire avec bonne API

**User:** "On veut connecter TheHive : https://docs.strangebee.com/thehive/api-docs/"

### Step 1: Research
- REST API bien documentée, auth = API key en header `Authorization: Bearer <key>`
- Endpoints : `/api/v1/case`, `/api/v1/alert`, `/api/v1/observable`
- Pas d'OpenAPI spec officiel mais API simple et cohérente

### Step 2: Strategy
API simple, pas de spec → **Path B** (manual)

### Step 3: Build

```
# Secret
tracecat_create_secret:
  name: "thehive"
  keys:
    - key: "API_KEY"
      value: "<user-provided>"
    - key: "BASE_URL"
      value: "https://thehive.internal:9000"

# Action: Create Alert in TheHive
tracecat_update_action:
  inputs: |
    url: ${{ SECRETS.thehive.BASE_URL }}/api/v1/alert
    method: POST
    headers:
      Authorization: Bearer ${{ SECRETS.thehive.API_KEY }}
      Content-Type: application/json
    payload:
      type: "tracecat-alert"
      source: "Tracecat SOAR"
      sourceRef: ${{ TRIGGER.data.alert_id }}
      title: ${{ TRIGGER.data.title }}
      description: ${{ TRIGGER.data.description }}
      severity: 2
      tlp: 2
      tags:
        - "tracecat"
        - "automated"
    verify_ssl: false

# Action: Search Cases
tracecat_update_action:
  inputs: |
    url: ${{ SECRETS.thehive.BASE_URL }}/api/v1/query
    method: POST
    headers:
      Authorization: Bearer ${{ SECRETS.thehive.API_KEY }}
    payload:
      query:
        _name: "listCase"
        _and:
          - _field: "status"
            _value: "Open"
```
