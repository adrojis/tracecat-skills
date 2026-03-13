# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability, please report it responsibly:

1. **Do not** open a public GitHub issue
2. Use GitHub private vulnerability reporting: [Report a vulnerability](https://github.com/adrojis/tracecat-skills/security/advisories/new)

## What This Repo Contains

This repository contains **Claude Code skill files** (Markdown prompts, examples, and documentation). It does **not** contain executable code, credentials, or API keys.

Skill files may reference Tracecat expression syntax like `${{ SECRETS.xxx.API_KEY }}` — these are template placeholders evaluated at runtime by Tracecat, not real credentials.

## Supported Versions

| Version | Supported |
|---|---|
| 1.x | Yes |
