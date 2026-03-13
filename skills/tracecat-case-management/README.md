# tracecat-case-management

## Purpose
Expert guide for managing security cases in Tracecat — case lifecycle, status transitions, priority management, comments, and automated case workflows.

## Activation Triggers
- User creates or manages security cases
- User asks about case lifecycle (new → in_progress → resolved → closed)
- User builds case automation workflows
- User configures case fields (priority, malice, status, action)
- User adds comments to cases

## Files
| File | Description |
|------|-------------|
| `SKILL.md` | Main skill — case lifecycle, fields, MCP tools |
| `README.md` | This file — metadata and overview |
| `COMMON_MISTAKES.md` | Wrong vs right patterns for case management |
| `EXAMPLES.md` | Complete case management examples |

## Dependencies
- **tracecat-action-configuration** — core.cases.* action types
- **tracecat-mcp-tools-expert** — MCP tools for case CRUD
- **tracecat-workflow-patterns** — Workflow patterns that create cases

## Success Criteria
- Cases use native action types (core.cases.create_case, not reshape)
- Case status follows valid lifecycle transitions
- Priority and malice are valid enum values
- Payload contains relevant investigation data
- Comments document key decisions and findings
