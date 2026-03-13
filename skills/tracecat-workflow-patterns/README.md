# tracecat-workflow-patterns

## Purpose
Proven SOAR workflow patterns for Tracecat — alert triage, incident response, scheduled scans, data enrichment, notification routing, and design principles.

## Activation Triggers
- User designs or architects a new workflow
- User asks about workflow patterns for a use case
- User plans automation pipeline architecture
- User asks about node layout and graph design
- User builds security automation (SOC, IR, compliance)

## Files
| File | Description |
|------|-------------|
| `SKILL.md` | Main skill — 5 patterns, design principles, layout |
| `examples.md` | 3 complete YAML workflow examples |
| `README.md` | This file — metadata and overview |
| `COMMON_MISTAKES.md` | Wrong vs right patterns for workflow design |
| `EXAMPLES.md` | Additional workflow architecture examples |

## Dependencies
- **tracecat-action-configuration** — Action types used in patterns
- **tracecat-mcp-tools-expert** — MCP tools for building workflows
- **tracecat-integration-expert** — Integrations used in patterns
- **tracecat-case-management** — Case operations in patterns
- **tracecat-validation-debug** — Validating built workflows

## Success Criteria
- Workflow follows a proven pattern (triage, IR, enrichment, etc.)
- All actions connected with edges and positioned
- Error paths included for critical actions
- Idempotent design (safe to re-run)
- Modular (sub-workflows for complex logic)
