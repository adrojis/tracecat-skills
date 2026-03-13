# tracecat-integration-builder

Build Tracecat integrations to connect external APIs/tools. Two methods, same output.

## Purpose

Guide through building integrations via two paths:
- **Path A**: OpenAPI 3.0 spec → `openapi_to_template.py` → auto-generated YAML templates
- **Path B**: No spec → manual YAML templates (research API docs, write by hand)

Both produce YAML action templates using `core.http_request` + Tracecat secrets. No custom OAuth, no separate MCP servers needed.

## When to Trigger

- User wants to add a new vendor integration
- User has an OpenAPI spec to convert
- User asks about `openapi_to_template.py`
- User wants to connect an API that has no native Tracecat integration

## Key Reference

- OpenAPI Converter docs: https://docs.tracecat.com/integrations/openapi-converter
- Real example: HarfangLab EDR (Path B — 17 manual templates, alert triage workflow)
