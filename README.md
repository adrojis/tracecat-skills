# tracecat-skills

> Expert Claude Code skills for building Tracecat SOAR workflows using the [tracecat-mcp](https://github.com/adrojis/tracecat-mcp) MCP server.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![tracecat-mcp compatible](https://img.shields.io/badge/tracecat--mcp-compatible-green.svg)](https://github.com/adrojis/tracecat-mcp)

---

## What is this?

A collection of **5 Claude Code skills** that give Claude deep expertise in [Tracecat](https://github.com/TracecatHQ/tracecat), the open-source SOAR (Security Orchestration, Automation, and Response) platform.

When installed, Claude Code automatically activates the right skill based on your request -- no manual selection needed. Ask Claude to build a workflow, configure an action, set up an integration, or manage cases, and it will use battle-tested patterns and avoid common pitfalls.

## Why These Skills Exist

Building Tracecat workflows programmatically involves many non-obvious conventions:

- Action types use underscores (`core.http_request`), not dots (`core.http.request`)
- HTTP actions use `payload`, not `body`
- Action inputs must be YAML strings, not JSON objects
- Expressions use `${{ }}` with specific operator rules (`&&`/`||`, not `and`/`or`)
- The workflow lifecycle requires save, commit, activate, and webhook setup in the right order
- `run_if` does not support loop variables -- you need upstream filters instead
- Python scripts must be inside functions (no top-level `return`)

These skills encode all of these rules and more, so Claude gets it right the first time.

## The 5 Skills

### 1. tracecat-action-configuration

Configure Tracecat workflow actions correctly.

**Key features:**
- All core action types with full parameter reference (HTTP, transform, AI, Python, child workflows)
- Control flow patterns (if/else branching, loops, retry policies)
- Expression syntax reference (contexts, operators, typecasting)
- Common mistakes and fixes

**Activates when you:** configure actions, set up HTTP requests, write Python scripts, use expressions, set up loops or conditions.

### 2. tracecat-case-management

Create and manage Tracecat cases and incidents.

**Key features:**
- Case lifecycle and status flow
- Priority vs severity guidance
- 15 case action references
- Auto-case creation, dual notification, and Splunk alert patterns

**Activates when you:** create cases, manage incidents, update case status, add comments, build case automation.

### 3. tracecat-mcp-tools-expert

Expert guide for using Tracecat MCP tools and API.

**Key features:**
- Quick reference: which MCP tool to use for each goal
- Workflow deployment sequence (create, link, commit, activate, trigger)
- Graph API usage for edges and node positioning
- API authentication patterns

**Activates when you:** interact with Tracecat via MCP, create or deploy workflows, check execution results, manage schedules.

### 4. tracecat-secrets-integrations

Configure integrations and secrets.

**Key features:**
- 42 integration namespaces with exact secret names and keys
- Splunk complete reference (18 native actions + HTTP patterns for missing features)
- Custom integration development (YAML templates and Python UDFs)
- IOC extraction functions reference
- Multi-tenant secret patterns

**Activates when you:** connect integrations, configure secrets, set up Splunk/CrowdStrike/Okta/Slack/Jira, build custom integrations.

### 5. tracecat-workflow-patterns

Proven workflow patterns for security automation.

**Key features:**
- 7 complete workflow patterns (alert triage, enrichment, compliance, Splunk alert, Splunk notables, Splunk saved searches, AI-SOAR)
- Programmatic workflow creation via API/MCP
- Graph API edge and position management
- Architecture best practices

**Activates when you:** build workflows, design automation, create alert triage, set up enrichment pipelines, plan workflow architecture.

## Installation

### Manual Installation

Copy the skill folders to your Claude Code skills directory:

```bash
# Clone the repository
git clone https://github.com/adrojis/tracecat-skills.git

# Copy skills to Claude Code
cp -r tracecat-skills/skills/* ~/.claude/skills/
```

Your `~/.claude/skills/` directory should look like:

```
~/.claude/skills/
  tracecat-action-configuration/
    SKILL.md
    README.md
  tracecat-case-management/
    SKILL.md
    README.md
  tracecat-mcp-tools-expert/
    SKILL.md
    README.md
  tracecat-secrets-integrations/
    SKILL.md
    README.md
  tracecat-workflow-patterns/
    SKILL.md
    README.md
```

## Usage

Skills activate automatically based on your request. Just ask Claude Code naturally:

```
# Triggers tracecat-workflow-patterns
"Build a workflow that triages Splunk alerts and creates cases"

# Triggers tracecat-action-configuration
"Configure an HTTP request action to call the VirusTotal API"

# Triggers tracecat-secrets-integrations
"Set up the CrowdStrike integration with secrets"

# Triggers tracecat-case-management
"Create a case from this alert with high priority"

# Triggers tracecat-mcp-tools-expert
"Deploy this workflow and make it triggerable via webhook"
```

## Skills Work Together

When you ask Claude to "build a Splunk alert triage workflow with case creation", multiple skills collaborate:

1. **tracecat-workflow-patterns** provides the Splunk alert triage pattern (Pattern 4)
2. **tracecat-action-configuration** ensures each action is configured correctly
3. **tracecat-secrets-integrations** sets up the Splunk and VirusTotal secrets
4. **tracecat-case-management** handles the case creation and comment patterns
5. **tracecat-mcp-tools-expert** orchestrates the MCP tool calls to deploy everything

## Related Projects

- [tracecat-mcp](https://github.com/adrojis/tracecat-mcp) -- MCP server for Tracecat (44 tools)
- [Tracecat](https://github.com/TracecatHQ/tracecat) -- Open-source SOAR platform

## License

MIT -- see [LICENSE](LICENSE).
