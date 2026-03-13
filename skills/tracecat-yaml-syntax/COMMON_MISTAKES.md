# Common Mistakes — YAML Syntax & Expressions

## 1. Unclosed expressions
**Wrong:**
```yaml
url: https://api.example.com/${{ TRIGGER.data.ip
```
**Right:**
```yaml
url: https://api.example.com/${{ TRIGGER.data.ip }}
```
**Why:** Every `${{` must be closed with `}}`. Unclosed expressions cause parsing errors.

## 2. Wrong context name
**Wrong:**
```yaml
value: ${{ TRIGGER.ip }}
value: ${{ ACTION.enrich.result }}
value: ${{ SECRET.virustotal.API_KEY }}
```
**Right:**
```yaml
value: ${{ TRIGGER.data.ip }}          # TRIGGER.data.* for webhook payloads
value: ${{ ACTIONS.enrich.result }}     # ACTIONS (plural)
value: ${{ SECRETS.virustotal.API_KEY }} # SECRETS (plural)
```
**Why:** Context names must be exact: `TRIGGER`, `ACTIONS`, `SECRETS`, `FN`, `ENV`.

## 3. Missing .data in TRIGGER context
**Wrong:**
```yaml
src_ip: ${{ TRIGGER.src_ip }}
```
**Right:**
```yaml
src_ip: ${{ TRIGGER.data.src_ip }}
```
**Why:** Webhook payload data is nested under `TRIGGER.data.*`.

## 4. Missing .result in ACTIONS context
**Wrong:**
```yaml
score: ${{ ACTIONS.enrich.score }}
```
**Right:**
```yaml
score: ${{ ACTIONS.enrich.result.score }}
```
**Why:** Action outputs are under `.result`. The `.error` path is only for failed actions.

## 5. Python logical operators
**Wrong:**
```yaml
run_if: ${{ ACTIONS.a.result and ACTIONS.b.result }}
run_if: ${{ not ACTIONS.a.result or ACTIONS.b.result }}
```
**Right:**
```yaml
run_if: ${{ ACTIONS.a.result && ACTIONS.b.result }}
run_if: ${{ not ACTIONS.a.result || ACTIONS.b.result }}
```
**Why:** Use `&&` and `||` (C-style). `and`/`or` are not supported.

## 6. Equality with booleans
**Wrong:**
```yaml
run_if: ${{ ACTIONS.check.result == true }}
run_if: ${{ ACTIONS.check.result == false }}
```
**Right:**
```yaml
run_if: ${{ ACTIONS.check.result }}
run_if: ${{ not ACTIONS.check.result }}
```
**Why:** Tracecat uses truthy/falsy evaluation. Explicit `== true` can fail on non-boolean values.

## 7. YAML quoting issues
**Wrong:**
```yaml
# Colon in value breaks YAML parsing
url: https://api.example.com:8443/path
```
**Right:**
```yaml
url: "https://api.example.com:8443/path"
```
Or as MCP YAML string: `"url: \"https://api.example.com:8443/path\""`

**Why:** YAML interprets `:` followed by space as key-value separator. URLs with ports need quoting.

## 8. Expression in wrong position
**Wrong:**
```yaml
${{ ACTIONS.check.result.url }}:
  method: GET
```
**Right:**
```yaml
url: ${{ ACTIONS.check.result.url }}
method: GET
```
**Why:** Expressions are values, not keys. They can't be used as YAML keys.

## 9. Nested expression
**Wrong:**
```yaml
value: ${{ ${{ ACTIONS.x.result }} }}
```
**Right:**
```yaml
value: ${{ ACTIONS.x.result }}
```
**Why:** Expressions don't nest. One `${{ }}` wrapper is sufficient.

## 10. String concatenation in expressions
**Wrong:**
```yaml
message: ${{ "Alert: " + ACTIONS.x.result.title }}
```
**Right:**
```yaml
message: "Alert: ${{ ACTIONS.x.result.title }}"
```
Or use FN.format:
```yaml
message: ${{ FN.format("Alert: {}", ACTIONS.x.result.title) }}
```
**Why:** String concatenation with `+` is not supported inside expressions. Use string interpolation or `FN.format`.
