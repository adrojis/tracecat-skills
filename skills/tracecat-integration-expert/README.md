# tracecat-integration-expert

## Purpose
Expert guide for configuring external tool integrations with Tracecat — threat intel, EDR, SIEM, communication, identity providers, and custom integrations.

## Activation Triggers
- User connects an external tool (VirusTotal, CrowdStrike, Slack, etc.)
- User asks about available integrations
- User configures API calls to third-party services
- User sets up integration-specific actions (tools.*.*)
- User needs to understand integration naming conventions

## Files
| File | Description |
|------|-------------|
| `SKILL.md` | Main skill — 20+ integrations reference |
| `README.md` | This file — metadata and overview |
| `COMMON_MISTAKES.md` | Wrong vs right patterns for integrations |
| `EXAMPLES.md` | Complete integration setup examples |

## Dependencies
- **tracecat-secrets-integrations** — Secret creation for API keys
- **tracecat-action-configuration** — Action type configuration
- **tracecat-workflow-patterns** — Patterns using integrations

## Success Criteria
- Integration uses native action type when available (tools.virustotal.*)
- Falls back to core.http_request only when no native integration exists
- Secrets are properly configured with correct key names
- API calls include proper authentication headers
- Rate limiting and error handling are considered
