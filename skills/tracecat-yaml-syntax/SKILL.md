---
name: tracecat-yaml-syntax
description: Activate when users write or edit Tracecat workflow YAML definitions, action inputs, or expression syntax
---

# Tracecat YAML Syntax

You are an expert at writing Tracecat workflow definitions in YAML. Use this reference when creating or modifying workflow YAML.

## Basic Structure

```yaml
definition:
  title: My Workflow
  description: Workflow description
  entrypoint: ref: action_1
  triggers:
    - type: webhook
      ref: webhook_trigger
  actions:
    - ref: action_1
      action: core.transform.reshape
      args:
        value: "Hello, ${{ TRIGGER.data.name }}"
```

## Expressions

Tracecat uses `${{ }}` syntax for dynamic expressions:

- **Trigger data:** `${{ TRIGGER.data.field_name }}`
- **Action results:** `${{ ACTIONS.action_ref.result }}`
- **Nested access:** `${{ ACTIONS.action_ref.result.data.items }}`
- **Environment:** `${{ ENV.variable_name }}`
- **Secrets:** `${{ SECRETS.secret_name.key }}`
- **Functions:** `${{ FN.str.upper(TRIGGER.data.name) }}`

## Action Types

### Core Actions
- `core.transform.reshape` — Transform/reshape data
- `core.http.request` — Make HTTP requests
- `core.send_email` — Send emails
- `core.workflow.execute` — Run sub-workflow

### Integration Actions
Format: `tools.<integration>.<action>`
- `tools.virustotal.analyze_url`
- `tools.slack.post_message`
- `tools.crowdstrike.contain_host`

## Control Flow

### Conditional Execution
```yaml
- ref: check_severity
  action: core.transform.reshape
  args:
    value: ${{ TRIGGER.data.severity }}
  run_if: ${{ FN.str.lower(TRIGGER.data.type) == "alert" }}
```

### Loops
```yaml
- ref: process_items
  action: core.transform.reshape
  args:
    value: ${{ for var.item in ACTIONS.get_items.result.items }}
  for_each: ${{ ACTIONS.get_items.result.items }}
```

### Parallel Actions
Define multiple actions with the same parent to run them in parallel:
```yaml
- ref: enrich_vt
  action: tools.virustotal.analyze_url
  args:
    url: ${{ TRIGGER.data.url }}
  depends_on:
    - action_1

- ref: enrich_shodan
  action: tools.shodan.search
  args:
    query: ${{ TRIGGER.data.ip }}
  depends_on:
    - action_1
```

## Input Schema (expects)

```yaml
definition:
  title: Enrichment Workflow
  entrypoint: ref: step_1
  inputs:
    ioc_value:
      type: str
      description: The IOC to enrich
    ioc_type:
      type: str
      description: Type of IOC (ip, domain, hash)
      default: ip
```

## Tips

- Always validate YAML syntax before deploying
- Use descriptive `ref` names (e.g., `enrich_ip_virustotal` not `step_3`)
- Keep secrets in Tracecat's secret manager, reference via `${{ SECRETS.* }}`
- Test with small payloads before deploying to production

## Related Skills
- **tracecat-mcp-tools-expert** — MCP tool reference for creating/updating workflows
- **tracecat-workflow-patterns** — Design patterns and templates
- **tracecat-integration-expert** — Integration-specific YAML configurations
- **tracecat-validation-debug** — Debug YAML expression errors
- **tracecat-code-python** — Python script YAML configuration

## Reference Files
- [Common Mistakes](./COMMON_MISTAKES.md)
- [Examples](./EXAMPLES.md)
- [README](./README.md)
