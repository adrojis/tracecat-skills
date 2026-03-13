# Examples — Repo Sync

## Example 1: Push MCP Server Updates

```
# 1. Check what changed locally
cd mcp_server
git diff --stat

# 2. Review the files to sync
# Source: mcp_server/src/ → Repo: <owner>/tracecat-mcp

# 3. Push via GitHub MCP
mcp__my-github__push_files({
  owner: "<owner>",
  repo: "tracecat-mcp",
  branch: "main",
  message: "Add tracecat_autofix_workflow and tracecat_tools_documentation tools",
  files: [
    { path: "src/tools/docs.ts", content: "..." },
    { path: "src/tools/workflows.ts", content: "..." },
    { path: "src/tools/templates.ts", content: "..." },
    { path: "src/server.ts", content: "..." }
  ]
})
```

---

## Example 2: Push Skill Updates

```
# Source: tracecat_skills/skills/ → Repo: <owner>/tracecat-skills

mcp__my-github__push_files({
  owner: "<owner>",
  repo: "tracecat-skills",
  branch: "main",
  message: "Add README.md, COMMON_MISTAKES.md, EXAMPLES.md for all skills",
  files: [
    { path: "skills/tracecat-action-configuration/README.md", content: "..." },
    { path: "skills/tracecat-action-configuration/COMMON_MISTAKES.md", content: "..." },
    { path: "skills/tracecat-action-configuration/EXAMPLES.md", content: "..." },
    // ... repeat for all 10 skills
  ]
})
```

---

## Example 3: Check Before Sync

```
# List current files in repo
mcp__my-github__get_file_contents({
  owner: "<owner>",
  repo: "tracecat-mcp",
  path: "src/tools"
})

# Compare with local
ls mcp_server/src/tools/

# Identify new/modified files to push
```

---

## Example 4: Safety Checklist

Before every sync:
1. **No secrets?** — grep for API keys, passwords, tokens in files to push
2. **No dist/?** — only push source files
3. **Correct repo?** — MCP code to tracecat-mcp, skills to tracecat-skills
4. **Descriptive commit?** — "Add X feature" not "update"
5. **Diff reviewed?** — read the files before pushing
