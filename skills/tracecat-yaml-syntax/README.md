# tracecat-yaml-syntax

## Purpose
Expert guide for Tracecat YAML syntax — expressions (${{ }}), contexts (TRIGGER, ACTIONS, SECRETS, FN), and workflow definition format.

## Activation Triggers
- User writes YAML for workflow definitions
- User uses expression syntax (${{ }})
- User references actions, triggers, secrets, or functions
- User encounters expression syntax errors
- User asks about available built-in functions (FN.*)

## Files
| File | Description |
|------|-------------|
| `SKILL.md` | Main skill — YAML structure, expression syntax |
| `reference.md` | Complete workflow example and function reference |
| `README.md` | This file — metadata and overview |
| `COMMON_MISTAKES.md` | Wrong vs right patterns for YAML/expressions |
| `EXAMPLES.md` | Complete YAML expression examples |

## Dependencies
- **tracecat-action-configuration** — Action inputs that use expressions
- **tracecat-validation-debug** — Debugging expression errors

## Success Criteria
- Expressions properly formatted: ${{ CONTEXT.path }}
- All ${{ opened are closed with }}
- Correct context used (TRIGGER, ACTIONS, SECRETS, FN, ENV)
- Functions called with correct arguments
- YAML properly indented and valid
