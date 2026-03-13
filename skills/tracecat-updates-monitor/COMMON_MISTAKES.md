# Common Mistakes — tracecat-updates-monitor

## 1. Not checking memory state first
**Wrong:** Running all checks without reading `tracecat-versions.md`
**Right:** Always read the memory file first to know what changed since last check
**Why:** Without baseline, you can't identify what's new vs. already known

## 2. Only checking latest release
**Wrong:** `gh api repos/TracecatHQ/tracecat/releases/latest`
**Right:** Check last 10 releases and compare with last known version
**Why:** Multiple releases can happen between sessions (Tracecat releases frequently)

## 3. Ignoring pre-release/beta tags
**Wrong:** Filtering out beta releases
**Right:** ALL Tracecat releases are currently beta (1.0.0-beta.X) — they ARE the main releases
**Why:** Tracecat hasn't hit stable 1.0 yet, beta IS production

## 4. Not updating memory after check
**Wrong:** Reporting findings but not updating `tracecat-versions.md`
**Right:** Always update the memory file with latest version, date, and findings
**Why:** Next check needs the baseline to compute diffs

## 5. Over-alerting on irrelevant changes
**Wrong:** Flagging every commit as important
**Right:** Use the impact assessment rules (HIGH/MEDIUM/LOW) to prioritize
**Why:** Too many false alarms leads to alert fatigue

## 6. Missing the tables → row inventories migration
**Wrong:** Assuming our table/row API calls still work on new versions
**Right:** Specifically check if `workspace tables` endpoints have changed
**Why:** Beta.29 replaced "workspace management tables with row inventories" — this directly affects our MCP tools
