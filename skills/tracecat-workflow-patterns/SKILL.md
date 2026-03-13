---
name: tracecat-workflow-patterns
description: Activate when users design, architect, or plan SOAR workflow patterns and automation pipelines in Tracecat
---

# Tracecat Workflow Patterns

You are an expert at designing SOAR workflow patterns for Tracecat. Use these patterns as templates when building security automation workflows.

## Critical Rule: Always Prioritize Native Action Types

When building any workflow, ALWAYS check for and use native/official action types BEFORE falling back to generic actions:

| Operation | WRONG (generic) | RIGHT (native) |
|-----------|-----------------|-----------------|
| Create case | `core.transform.reshape` + manual POST | `core.cases.create_case` |
| Update case | `core.transform.reshape` + manual POST | `core.cases.update_case` |
| Add comment | `core.http_request` to /cases/comments | `core.cases.create_comment` |
| VirusTotal lookup | `core.http_request` with manual headers | `tools.virustotal.*` |
| CrowdStrike query | `core.http_request` with OAuth flow | `tools.crowdstrike.*` |
| Splunk search | `core.http_request` to Splunk API | `tools.splunk.*` |
| Defender alerts | `core.http_request` with Azure auth | `tools.microsoft_defender.*` |
| SharePoint files | `core.http_request` with Graph API | `tools.sharepoint.*` |
| Okta user mgmt | `core.http_request` to Okta API | `tools.okta.*` |

**Fallback rules:**
- `core.http_request` → ONLY when no native integration exists for that API
- `core.transform.reshape` → ONLY for data transformation/aggregation, never for CRUD operations
- `core.script.run_python` → ONLY when logic cannot be expressed with native actions or expressions

## Pattern 1: Alert Triage

**Use case:** Automatically triage incoming security alerts.

```
Webhook trigger → Enrich alert (VirusTotal, AbuseIPDB) → Score risk → Decision:
  - High risk → Create case + Notify SOC (Slack)
  - Medium risk → Create case
  - Low risk → Log and close
```

**Key actions:** `tools.virustotal.analyze_url`, `tools.abuseipdb.check_ip`, `core.transform.reshape`

## Pattern 2: Incident Response

**Use case:** Automated incident response playbook.

```
Case created → Gather context (SIEM query) → Containment:
  - Block IP (firewall)
  - Disable user (IAM)
  - Isolate host (EDR)
→ Document actions → Notify stakeholders
```

**Key actions:** `tools.crowdstrike.contain_host`, `core.http.request`

## Pattern 3: Scheduled Scan

**Use case:** Periodic security scanning and reporting.

```
Cron trigger (daily) → Scan targets → Compare with baseline:
  - New findings → Create cases
  - Resolved findings → Close cases
→ Generate report → Email/Slack
```

**Key actions:** `core.workflow.execute` (sub-workflow), `tools.slack.post_message`

## Pattern 4: Data Enrichment Pipeline

**Use case:** Enrich IOCs from multiple sources.

```
Input IOC → Parallel lookup:
  - VirusTotal
  - Shodan
  - GreyNoise
  - AbuseIPDB
→ Merge results → Score → Store in table
```

**Key actions:** Use parallel execution with `core.transform.reshape` to merge

## Pattern 5: Notification Pipeline

**Use case:** Smart notification routing based on severity.

```
Trigger → Evaluate severity:
  - Critical → PagerDuty + Slack #critical + Email
  - High → Slack #alerts + Email
  - Medium → Slack #alerts
  - Low → Log only
```

**Key actions:** `tools.slack.post_message`, `core.http.request` (PagerDuty API)

## Design Principles

1. **Idempotency** — Workflows should be safe to re-run
2. **Error handling** — Always add fallback paths for action failures
3. **Deduplication** — Check if a case/alert already exists before creating
4. **Audit trail** — Log key decisions with case comments
5. **Modularity** — Break complex workflows into sub-workflows
6. **Node layout** — Always position nodes after creation (see below)

## Node Layout Guidelines

When building workflows programmatically, **always reposition nodes**. MCP-created nodes default to (0,0) and overlap.

### Dimensions and Constants

| Element | Width | Height | Notes |
|---------|-------|--------|-------|
| Trigger node | ~280px | ~200px | Taller than actions |
| Action node | ~250px | ~100px | Standard UDF node |
| Viewport default | - | - | Best centered at x=500 |

### Spacing Rules

| Rule | Value | Why |
|------|-------|-----|
| **Trigger position** | **(500, 0)** — always | Top-center anchor point |
| **First action Y** | **y >= 300** | Trigger is ~200px tall, leave 100px margin |
| **Vertical gap** | **200px** between levels | Nodes ~100px tall, 200px gives breathing room |
| **Horizontal gap** | **350px minimum** between parallel siblings | Nodes ~250px wide, avoids overlap |
| **Main spine X** | **x = 500** | Center of viewport, aligns with trigger |

### Layout Patterns with Exact Coordinates

**Linear chain** (most common):
```
Trigger:    (500, 0)
Action 1:   (500, 300)     ← first action at y=300
Action 2:   (500, 500)     ← +200
Action 3:   (500, 700)     ← +200
Action 4:   (500, 900)     ← +200
```

**Fan-out / fan-in (2 parallel branches)**:
```
Trigger:       (500, 0)
Parent:        (500, 300)
                 ┌──┴──┐
Left branch:  (200, 500)    Right branch: (800, 500)
                 └──┬──┘
Merge:         (500, 700)
Next:          (500, 900)
```
- Siblings offset ±300 from center (200 and 800 = 600px apart > 350px minimum)
- Merge node centered below at same x as parent

**Fan-out / fan-in (3 parallel branches)**:
```
Trigger:          (500, 0)
Parent:           (500, 300)
              ┌─────┼─────┐
Left:       (150, 500)  Center: (500, 500)  Right: (850, 500)
              └─────┼─────┘
Merge:            (500, 700)
```
- 350px between each sibling (150 → 500 → 850)

**Decision / conditional branches (success + error)**:
```
Trigger:       (500, 0)
Action:        (500, 300)
         ─success─┘ └─error─
Success path: (300, 500)    Error path: (700, 500)
              └──────┬──────┘
Final:         (500, 700)    ← join_strategy: any
```

**Diamond pattern (split then merge)**:
```
Trigger:       (500, 0)
Fetch data:    (500, 300)
           ┌────┴────┐
Enrich A: (200, 500) Enrich B: (800, 500)
           └────┬────┘
Score:     (500, 700)   ← depends on both, join_strategy: all
Decision:  (500, 900)
```

### Positioning Anti-Patterns

| Anti-pattern | Problem | Fix |
|-------------|---------|-----|
| All nodes at (0, 0) | Everything overlaps, invisible graph | Always call `move_nodes` after creation |
| Trigger not at (500, 0) | Trigger misaligned with actions | Always call `update_trigger_position` |
| Parallel nodes too close | Overlap when <350px apart | Use ±300 offset from center minimum |
| No vertical margin after trigger | Action hides behind trigger | First action at y >= 300 |
| Inconsistent X for linear chain | Zigzag layout, hard to read | Same X for all linear nodes |

### MCP Tool Calls

Always execute these 3 calls after creating actions:
```
1. tracecat_add_edges(...)               # Connect all nodes
2. tracecat_move_nodes(...)              # Position action nodes
3. tracecat_update_trigger_position(...) # Position trigger at (500, 0)
```

Format for `move_nodes`:
`{"positions": [{"action_id": "uuid", "x": 500, "y": 300}, ...]}`

See `tracecat-mcp-tools-expert` skill for full Graph API details.

## Related Skills
- **tracecat-mcp-tools-expert** — MCP tool reference and Graph API operations
- **tracecat-yaml-syntax** — YAML syntax for workflow definitions
- **tracecat-integration-expert** — Configure external tool integrations
- **tracecat-case-management** — Case lifecycle patterns
- **tracecat-validation-debug** — Debug and validate workflows
- **tracecat-code-python** — Python scripting for data processing actions

## Reference Files
- [Common Mistakes](./COMMON_MISTAKES.md)
- [Examples](./examples.md)
- [README](./README.md)
