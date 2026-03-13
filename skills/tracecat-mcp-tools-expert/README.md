# tracecat-mcp-tools-expert

## Purpose
Master guide for using all Tracecat MCP tools — 50+ tools across workflows, actions, executions, cases, secrets, tables, schedules, graph, and documentation.

## Activation Triggers
- User interacts with Tracecat via MCP tools
- User asks which tool to use for a specific operation
- User builds workflows programmatically
- User manages graph layout (edges, positions)
- User queries executions or cases

## Files
| File | Description |
|------|-------------|
| `SKILL.md` | Main skill — all MCP tools reference, Graph API |
| `README.md` | This file — metadata and overview |
| `COMMON_MISTAKES.md` | Wrong vs right patterns for MCP tool usage |
| `EXAMPLES.md` | Complete MCP tool usage sequences |

## Dependencies
- **tracecat-action-configuration** — Action update specifics
- **tracecat-workflow-patterns** — Workflow creation patterns
- **tracecat-validation-debug** — Validation tool usage

## Success Criteria
- Correct tool selected for the operation
- Required parameters provided
- Inputs passed as YAML strings (not JSON objects)
- Graph operations include edges AND positions
- Workspace ID auto-injected (no manual handling needed)
