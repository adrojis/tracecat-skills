# Contributing to tracecat-skills

Thanks for your interest in contributing! Here's how to get started.

## Structure

Each skill lives in `skills/tracecat-<name>/` with these files:

| File | Purpose |
|---|---|
| `SKILL.md` | Main prompt loaded by Claude Code — this is what the AI reads |
| `README.md` | Human-readable documentation |
| `COMMON_MISTAKES.md` | Pitfall catalog with fixes |
| `EXAMPLES.md` | Real-world usage examples |

## Adding a New Skill

1. Create `skills/tracecat-<name>/` with all 4 files
2. Add a `## Reference Files` section at the bottom of `SKILL.md` linking to the companion files
3. Update the root `README.md` to list your new skill
4. Open a PR against `main`

## Improving an Existing Skill

The most impactful contributions are:

- **New entries in COMMON_MISTAKES.md** — if you hit a pitfall that isn't documented, add it
- **New entries in EXAMPLES.md** — real-world patterns that others can learn from
- **Corrections** — if a skill gives wrong advice, fix it

## Guidelines

- Keep skill prompts concise — Claude Code loads them into context, so every token counts
- Use concrete examples over abstract explanations
- Include the "why" alongside the "what" in common mistakes
- Test your changes by installing the skill locally and verifying Claude Code activates it correctly

## Reporting Issues

- Use [GitHub Issues](https://github.com/adrojis/tracecat-skills/issues)
- Include your Tracecat version
- Describe what Claude Code did wrong and what you expected
