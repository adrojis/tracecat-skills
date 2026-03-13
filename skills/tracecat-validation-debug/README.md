# tracecat-validation-debug

## Purpose
Expert guide for debugging failed workflow executions and validating workflows — error types, debug procedures, retry policies, and error paths.

## Activation Triggers
- User encounters a workflow execution error
- User wants to validate a workflow before deployment
- User asks about error types or debugging steps
- User configures retry policies or error paths
- User needs to interpret execution results

## Files
| File | Description |
|------|-------------|
| `SKILL.md` | Main skill — error types, debug procedure, retry policies |
| `error-catalog.md` | 20+ specific error examples with causes and fixes |
| `README.md` | This file — metadata and overview |
| `COMMON_MISTAKES.md` | Wrong vs right patterns for debugging |
| `EXAMPLES.md` | Complete debugging scenario examples |

## Dependencies
- **tracecat-mcp-tools-expert** — MCP tools for validation and execution inspection
- **tracecat-action-configuration** — Action configuration that causes errors
- **tracecat-yaml-syntax** — Expression syntax errors

## Success Criteria
- Error correctly identified and categorized
- Root cause found (not just symptoms)
- Fix applied and verified via re-execution
- Retry policies configured when appropriate
- Error paths set up for graceful failure handling
