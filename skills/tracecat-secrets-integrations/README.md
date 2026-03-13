# tracecat-secrets-integrations

## Purpose
Expert guide for configuring secrets and connecting external tools with authentication in Tracecat — secret types, key naming, and 16+ integration-specific configurations.

## Activation Triggers
- User creates or manages secrets
- User connects a new integration requiring authentication
- User references secrets in workflow expressions
- User asks about secret types or key naming
- User troubleshoots secret access errors

## Files
| File | Description |
|------|-------------|
| `SKILL.md` | Main skill — secret management, 16+ integrations |
| `README.md` | This file — metadata and overview |
| `COMMON_MISTAKES.md` | Wrong vs right patterns for secrets |
| `EXAMPLES.md` | Complete secret setup and usage examples |

## Dependencies
- **tracecat-integration-expert** — Integration requirements
- **tracecat-mcp-tools-expert** — MCP tools for secret CRUD
- **tracecat-action-configuration** — Using secrets in action inputs

## Success Criteria
- Secret name follows convention (lowercase, provider name)
- Key names match what integrations expect
- Secrets accessed via ${{ SECRETS.name.KEY }}
- No hardcoded credentials in workflows
- Secret type is appropriate (custom, token, oauth2)
