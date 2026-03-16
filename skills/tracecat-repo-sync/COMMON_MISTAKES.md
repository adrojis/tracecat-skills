# Common Mistakes — Repo Sync

## 1. Pushing .env or credentials
**Wrong:**
```
git add -A  # Includes .env, .env.local
```
**Right:**
```
# Only push specific files/directories
git add src/ package.json tsconfig.json
```
**Why:** `.env` files contain API keys and passwords. Never push them to GitHub.

## 2. Pushing dist/ or node_modules/
**Wrong:**
```
git add dist/ node_modules/
```
**Right:**
Ensure `.gitignore` excludes:
```
dist/
node_modules/
.env
.env.local
```
**Why:** Compiled JS and dependencies are generated artifacts. They bloat the repo and cause conflicts.

## 3. Wrong repository target
**Wrong:**
```
# Pushing MCP server code to tracecat-skills repo
```
**Right:**
- MCP Server code → `<owner>/tracecat-mcp-community`
- Skills files → `<owner>/tracecat-skills`
**Why:** Each codebase has its own repository. Mixing them creates confusion.

## 4. Vague commit messages
**Wrong:**
```
git commit -m "update"
git commit -m "fix"
```
**Right:**
```
git commit -m "Add tracecat_autofix_workflow tool for auto-fixing orphan nodes"
git commit -m "Add COMMON_MISTAKES.md for all 10 skills"
```
**Why:** Descriptive commits help track changes and understand history.

## 5. Not reviewing diff before push
**Wrong:**
```
git push  # Without reviewing
```
**Right:**
```
git diff --stat  # Review what changed
git push
```
**Why:** Always verify you're pushing the intended changes, not accidental modifications.
