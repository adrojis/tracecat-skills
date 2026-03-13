# Tracecat Skills - Development Guide

## Structure

```
skills/                          # 12 Claude Code skills
  tracecat-<name>/
    SKILL.md                     # Main skill prompt (loaded by Claude Code)
    README.md                    # Human-readable documentation
    COMMON_MISTAKES.md           # Pitfall catalog with fixes
    EXAMPLES.md                  # Real-world usage examples
integrations/                    # Action templates for specific platforms
  harfanglab/                    # 17 HarfangLab EDR templates
.claude-plugin/
  plugin.json                    # Plugin metadata
  marketplace.json               # Marketplace listing
```

## Skills

| Skill | Domain |
|---|---|
| tracecat-mcp-tools-expert | MCP tool usage, workflow deployment |
| tracecat-workflow-patterns | Workflow architecture, patterns |
| tracecat-action-configuration | Action types, inputs, control flow |
| tracecat-case-management | Cases, incidents, comments |
| tracecat-secrets-integrations | Secrets, 42 integration namespaces |
| tracecat-integration-builder | OpenAPI → action template conversion |
| tracecat-integration-expert | Platform-specific integration details |
| tracecat-yaml-syntax | Expression syntax, functions, operators |
| tracecat-validation-debug | Error catalog, debugging |
| tracecat-code-python | Python scripts for run_python actions |
| tracecat-updates-monitor | Release and API change tracking |
| tracecat-repo-sync | GitHub sync workflow |

## Adding a Skill

1. Create `skills/tracecat-<name>/SKILL.md` with the skill prompt
2. Add `README.md`, `COMMON_MISTAKES.md`, `EXAMPLES.md`
3. Reference companion files in SKILL.md's `## Reference Files` section
