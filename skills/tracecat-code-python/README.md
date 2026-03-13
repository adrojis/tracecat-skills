# tracecat-code-python

## Purpose
Expert guide for writing Python scripts in Tracecat's `core.script.run_python` action — function structure, input handling, dependency management, and common patterns.

## Activation Triggers
- User writes Python code for a Tracecat action
- User asks about run_python configuration
- User needs data processing, filtering, or transformation in Python
- User needs external API calls from Python (allow_network)
- User asks about available Python modules

## Files
| File | Description |
|------|-------------|
| `SKILL.md` | Main skill — script structure, inputs, rules |
| `examples.md` | 6 detailed Python script examples |
| `README.md` | This file — metadata and overview |
| `COMMON_MISTAKES.md` | Wrong vs right patterns for Python scripts |
| `EXAMPLES.md` | Additional real-world Python examples |

## Dependencies
- **tracecat-action-configuration** — Action type and inputs format
- **tracecat-yaml-syntax** — Expression syntax for passing data to scripts
- **tracecat-validation-debug** — Debugging Python script errors

## Success Criteria
- Script has a `main()` function that returns JSON-serializable data
- Field name is `script` (not `code`)
- Complex objects passed via `FN.serialize_json()` in inputs
- `allow_network: true` set when script makes HTTP calls
- Dependencies listed for non-stdlib packages
- Timeout set appropriately for long-running scripts
