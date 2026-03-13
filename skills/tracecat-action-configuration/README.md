# tracecat-action-configuration

## Purpose
Expert guide for configuring Tracecat workflow actions correctly — action types, inputs format, control flow (run_if, for_each, retry), and field naming conventions.

## Activation Triggers
- User creates or updates a workflow action
- User asks about action types (core.http_request, core.transform.reshape, etc.)
- User configures control flow (conditions, loops, retries)
- User writes YAML inputs for an action
- User encounters action configuration errors

## Files
| File | Description |
|------|-------------|
| `SKILL.md` | Main skill — action types reference, control flow, MCP usage |
| `README.md` | This file — metadata and overview |
| `COMMON_MISTAKES.md` | Wrong vs right patterns for action config |
| `EXAMPLES.md` | Complete action configuration examples |

## Dependencies
- **tracecat-yaml-syntax** — Expression syntax used in inputs
- **tracecat-code-python** — Python script action details
- **tracecat-mcp-tools-expert** — MCP tools for creating/updating actions
- **tracecat-case-management** — Case action types
- **tracecat-secrets-integrations** — Secret references in inputs

## Success Criteria
- Action type is correct (underscores, not dots)
- Inputs are YAML strings (not JSON objects) when using MCP
- Boolean expressions use truthy/falsy (not == true/false)
- Logical operators use && / || (not and/or)
- Python scripts use `script` field (not `code`)
- Control flow expressions are properly formatted
