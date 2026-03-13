# tracecat-updates-monitor

Monitor Tracecat GitHub releases, documentation, and API changes to keep our MCP server and skills up-to-date.

## Purpose

Tracecat is actively developed (multiple releases per week). This skill:
- Detects new releases and breaking changes
- Watches for an official Tracecat MCP server that could overlap with ours
- Tracks API endpoint changes that affect our tools
- Monitors documentation for new features/integrations

## Usage

Invoke with `/tracecat-updates-monitor` or ask Claude to "check for Tracecat updates".

## What It Checks

1. **GitHub Releases** — version diffs, changelogs, breaking changes
2. **Recent Commits** — MCP, API, registry, table/row changes
3. **Documentation** — new pages, structural changes, API reference updates
4. **Official MCP Server** — NPM, PyPI, GitHub searches
5. **Competition** — Tracecat's native `/mcp` endpoint evolution

## State Persistence

Stores last-checked state in memory file `tracecat-versions.md` to enable diff-based reporting across sessions.

## Dependencies

- `gh` CLI (GitHub API access)
- `WebFetch` / `WebSearch` tools (docs monitoring)
- Memory system (state persistence)
