# tracecat-skills

**Expert Claude Code skills for building flawless Tracecat SOAR workflows using the [tracecat-mcp](https://github.com/adrojis/tracecat-mcp) MCP server.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![tracecat-mcp](https://img.shields.io/badge/tracecat--mcp-compatible-green.svg)](https://github.com/adrojis/tracecat-mcp)
[![Tracecat](https://img.shields.io/badge/Tracecat-v1.0.0--beta.31-purple.svg)](https://github.com/TracecatHQ/tracecat)

---

## What is this?

A collection of **12 Claude Code skills** that give Claude deep expertise in [Tracecat](https://github.com/TracecatHQ/tracecat), the open-source SOAR (Security Orchestration, Automation, and Response) platform.

When installed, Claude Code automatically activates the right skill based on your request — no manual selection needed. Ask Claude to build a workflow, configure an action, set up an integration, or manage cases, and it will use battle-tested patterns and avoid common pitfalls.

### Why These Skills Exist

Building Tracecat workflows programmatically involves many non-obvious conventions:

- Action types use underscores (`core.http_request`), not dots (`core.http.request`)
- HTTP actions use `payload`, not `body`
- Action inputs must be YAML strings, not JSON objects
- Expressions use `${{ }}` with specific operator rules (`&&`/`||`, not `and`/`or`)
- The workflow lifecycle requires save, commit, activate, and webhook setup in the right order
- `run_if` does not support loop variables — you need upstream filters instead
- Python scripts must be inside functions (no top-level `return`)

These skills encode all of these rules and more, so Claude gets it right the first time.

---

## The 12 Skills

### 1. tracecat-mcp-tools-expert (HIGHEST PRIORITY)

Expert guide for using Tracecat MCP tools and API effectively.

**Activates when:** interacting with Tracecat via MCP, deploying workflows, checking executions, managing schedules.

**Key features:**
- Quick reference: which MCP tool to use for each goal
- Workflow deployment sequence (create, link, commit, activate, trigger)
- Graph API usage for edges and node positioning
- API authentication patterns

---

### 2. tracecat-workflow-patterns

Proven workflow patterns for security automation.

**Activates when:** building workflows, designing automation, creating alert triage, enrichment pipelines, planning architecture.

**Key features:**
- 7 complete workflow patterns (alert triage, enrichment, compliance, Splunk alert, Splunk notables, Splunk saved searches, AI-SOAR)
- Programmatic workflow creation via API/MCP
- Graph API edge and position management
- Node layout guidelines with exact coordinates

---

### 3. tracecat-action-configuration

Configure Tracecat workflow actions correctly.

**Activates when:** configuring actions, setting up HTTP requests, writing expressions, using loops or conditions.

**Key features:**
- All core action types with full parameter reference (HTTP, transform, AI, Python, child workflows)
- Control flow patterns (if/else branching, loops, retry policies)
- Expression syntax reference (contexts, operators, typecasting)
- Common mistakes catalog

---

### 4. tracecat-case-management

Create and manage Tracecat cases and incidents.

**Activates when:** creating cases, managing incidents, updating status, adding comments, building case automation.

**Key features:**
- Case lifecycle and status flow
- Priority vs severity guidance
- 15 case action references
- Auto-case creation, dual notification, and Splunk alert patterns

---

### 5. tracecat-secrets-integrations

Configure integrations and secrets.

**Activates when:** connecting integrations, configuring secrets, setting up Splunk/CrowdStrike/Okta/Slack/Jira.

**Key features:**
- 42 integration namespaces with exact secret names and keys
- Splunk complete reference (18 native actions + HTTP patterns)
- Custom integration development (YAML templates and Python UDFs)
- IOC extraction functions reference

---

### 6. tracecat-integration-builder

Build custom Tracecat integrations from OpenAPI specs.

**Activates when:** creating new integrations, converting OpenAPI specs, building action templates.

**Key features:**
- Full OpenAPI-to-template conversion workflow
- Config YAML reference (filtering, overrides, auth injection)
- Namespace and naming conventions
- End-to-end examples (CrowdStrike, Splunk, HarfangLab)

---

### 7. tracecat-integration-expert

Deep integration knowledge for specific platforms.

**Activates when:** debugging integration issues, configuring advanced API calls, handling auth edge cases.

**Key features:**
- Platform-specific gotchas (VirusTotal base64url, CrowdSec auth, MISP)
- HTTP action patterns for APIs without native integration
- SSL/TLS and timeout configuration
- Wazuh, CrowdSec, MISP integration patterns

---

### 8. tracecat-yaml-syntax

YAML expression syntax for Tracecat workflows.

**Activates when:** writing YAML inputs, using expressions, referencing action results, using built-in functions.

**Key features:**
- Expression context reference (`ACTIONS`, `TRIGGER`, `SECRETS`, `ENV`, `FN`, `INPUTS`)
- Operator rules and typecasting
- Built-in functions (serialize_json, deserialize_json, to_base64url, now, etc.)
- Common YAML pitfalls and fixes

---

### 9. tracecat-validation-debug

Debug workflow validation errors and execution failures.

**Activates when:** encountering validation errors, debugging failed executions, fixing expression issues.

**Key features:**
- Error catalog with solutions
- Validation error interpretation
- Execution failure diagnosis
- Common entrypoint and expression errors

---

### 10. tracecat-code-python

Write Python scripts for Tracecat `run_python` actions.

**Activates when:** writing Python scripts, using `core.script.run_python`, processing data in workflows.

**Key features:**
- Script structure requirements (must use functions, no top-level return)
- Input/output patterns
- Available libraries and sandbox limitations
- `FN.serialize_json` pattern for complex outputs

---

### 11. tracecat-updates-monitor

Track Tracecat releases and API changes.

**Activates when:** checking for updates, reviewing changelog, comparing versions.

**Key features:**
- GitHub release monitoring
- API change tracking
- Breaking change detection
- Migration guidance

---

### 12. tracecat-repo-sync

Synchronize local code to GitHub repositories.

**Activates when:** explicitly asked to sync changes to GitHub.

**Key features:**
- Safe push workflow with diff review
- Branch management
- Conflict resolution patterns

---

## Installation

### Prerequisites

1. **[tracecat-mcp](https://github.com/adrojis/tracecat-mcp)** MCP server installed and configured
2. **Claude Code** CLI installed
3. A running Tracecat instance

### Install Skills

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
    COMMON_MISTAKES.md
    EXAMPLES.md
  tracecat-case-management/
    ...
  tracecat-mcp-tools-expert/
    ...
  (... 12 skill directories total)
```

### Install Integration Templates (Optional)

The `integrations/` folder contains ready-made action templates for specific platforms:

```bash
# Example: install HarfangLab EDR templates
cp -r tracecat-skills/integrations/harfanglab/ /path/to/your/tracecat/templates/
```

---

## Usage

Skills activate **automatically** based on your request. Just ask Claude Code naturally:

```
"Build a workflow that triages Splunk alerts and creates cases"
→ Activates: tracecat-workflow-patterns + tracecat-action-configuration

"Configure an HTTP request action to call the VirusTotal API"
→ Activates: tracecat-action-configuration + tracecat-integration-expert

"Set up the CrowdStrike integration with secrets"
→ Activates: tracecat-secrets-integrations

"Create a case from this alert with high priority"
→ Activates: tracecat-case-management

"Deploy this workflow and make it triggerable via webhook"
→ Activates: tracecat-mcp-tools-expert

"Build a custom integration from this OpenAPI spec"
→ Activates: tracecat-integration-builder
```

### Skills Work Together

When you ask: **"Build a Splunk alert triage workflow with case creation"**

1. **tracecat-workflow-patterns** provides the Splunk alert triage pattern
2. **tracecat-action-configuration** ensures each action is configured correctly
3. **tracecat-secrets-integrations** sets up the Splunk and VirusTotal secrets
4. **tracecat-case-management** handles case creation and comment patterns
5. **tracecat-mcp-tools-expert** orchestrates the MCP tool calls to deploy everything

All skills compose seamlessly.

---

## What's Included

- **12** complementary skills that work together
- **17** HarfangLab EDR integration templates
- **COMMON_MISTAKES.md** for each skill — catalog of pitfalls with fixes
- **EXAMPLES.md** for each skill — real-world usage examples
- **Error catalog** for validation and debugging

---

## Roadmap

This project is under active development and will continue to evolve alongside Tracecat. As the platform ships new features, action types, and API changes, we update the skills to match.

Planned areas of improvement:

- **More integrations** — action templates for CrowdStrike, Wazuh, MISP, TheHive, and others
- **More skills** — dedicated skills for advanced topics (child workflows, multi-tenant setups, compliance automation)
- **Evaluation suite** — automated quality checks for each skill
- **Claude.ai / API support** — installation guides beyond Claude Code CLI
- **Community patterns** — curated workflow templates contributed by users

Contributions, issues, and feature requests are welcome.

---

## Related Projects

- [tracecat-mcp](https://github.com/adrojis/tracecat-mcp) — MCP server for Tracecat (49 tools)
- [Tracecat](https://github.com/TracecatHQ/tracecat) — Open-source SOAR platform

---

## License

[MIT](LICENSE)
