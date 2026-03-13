# Common Mistakes — Case Management

## 1. Using reshape instead of native case actions
**Wrong:**
```yaml
type: core.transform.reshape
inputs:
  value:
    case_title: "Alert"
    status: "new"
# Then calling /cases API manually
```
**Right:**
```yaml
type: core.cases.create_case
inputs:
  case_title: "Alert detected"
  status: new
  priority: high
  malice: unknown
```
**Why:** Native `core.cases.*` actions handle workspace association, validation, and proper field formatting.

## 2. Invalid status value
**Wrong:**
```yaml
status: open
status: active
status: pending
```
**Right:**
```yaml
status: new           # Initial state
status: in_progress   # Under investigation
status: resolved      # Fixed/mitigated
status: closed        # Closed out
```
**Why:** Only 4 valid status values. Anything else causes a validation error.

## 3. Invalid priority value
**Wrong:**
```yaml
priority: 1
priority: urgent
priority: p1
```
**Right:**
```yaml
priority: low
priority: medium
priority: high
priority: critical
```
**Why:** Priority must be one of these 4 string values.

## 4. Invalid malice value
**Wrong:**
```yaml
malice: suspicious
malice: true
malice: safe
```
**Right:**
```yaml
malice: benign
malice: malicious
malice: unknown
```
**Why:** Only 3 valid malice levels.

## 5. Missing workflow_id on case creation
**Wrong:**
```
tracecat_create_case({
  case_title: "Alert"
})
```
**Right:**
```
tracecat_create_case({
  workflow_id: "wf_xxx",
  case_title: "Alert"
})
```
**Why:** Every case must be associated with a workflow. `workflow_id` is required.

## 6. Empty payload
**Wrong:**
```yaml
case_title: "Suspicious activity"
# No payload
```
**Right:**
```yaml
case_title: "Suspicious activity from 1.2.3.4"
payload:
  src_ip: ${{ TRIGGER.data.src_ip }}
  detection_rule: ${{ TRIGGER.data.rule_name }}
  vt_score: ${{ ACTIONS.enrich.result.score }}
```
**Why:** The payload carries all investigation context. Empty payloads make cases useless for analysts.

## 7. Hardcoded case_id in comments
**Wrong:**
```yaml
type: core.cases.create_comment
inputs:
  case_id: "case_abc123"
  content: "Status update"
```
**Right:**
```yaml
type: core.cases.create_comment
inputs:
  case_id: ${{ ACTIONS.create_case.result.id }}
  content: "Status update"
```
**Why:** Case IDs should come from the case creation action result, not hardcoded.

## 8. Updating case with PATCH method
**Wrong (via raw API):**
```
PATCH /cases/{id}
```
**Right (via MCP):**
```
tracecat_update_case({ case_id: "...", status: "resolved" })
```
**Why:** The MCP tool handles the correct HTTP method. If using raw API, note the behavior may vary by version.
