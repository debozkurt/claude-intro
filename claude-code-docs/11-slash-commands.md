# Chapter 11 — Slash Commands

*Last verified: 2026-04-19 — Prerequisites: Ch 01 — Status: Extensions*

**Fresh ground** — no prior `agents/` coverage of the slash-command primitive.

---

## Concept

A slash command is a **user-invoked prompt template**, stored as a markdown file [1]. Type `/<name>` in the session; Claude receives the file's contents as the next prompt (with any args substituted). That's it. No tool restrictions, no model matching, no lifecycle events.

Slash commands are the simplest of the four extension primitives — and the right answer more often than people think. If you find yourself typing the same prompt twice, it belongs as a slash command.

## How it works

### Location

```
~/.claude/commands/<name>.md              # user-level, all projects
<repo>/.claude/commands/<name>.md         # project-level, committed
```

Subdirectories work — `.claude/commands/review/pr.md` becomes `/review:pr` in the session.

### Anatomy

A command file is just markdown — frontmatter is optional:

```markdown
---
description: Summarize the changes in my current branch
argument-hint: "[base-branch]"
---

Diff my current branch against $ARGUMENTS (default: main). Give me:
- A 2-line summary of the change
- Files touched
- Anything that would concern a reviewer
```

The `$ARGUMENTS` placeholder gets the text the user typed after `/<name>`.

Invocation:

```
/branch-summary feature/auth
```

### What makes it different from a skill

| | Slash command | Skill |
|---|---|---|
| Invocation | User types `/name` only | User or model (auto-suggested) |
| Tool budget | Whatever the session already has | Declared in frontmatter |
| Progressive disclosure | No — file loads only when invoked | Yes — frontmatter always, body on invoke |
| Complexity | Single prompt template | Procedure with phases, modes, flags |

Slash commands are thinner and simpler. When you don't need model-invocation, tool restrictions, or multi-phase logic, a slash command is the right answer.

## Why it matters

Slash commands are where most Claude Code productivity gains actually come from for active users. Three reasons:

1. **Zero-overhead shortcuts** — every repeated prompt is a candidate. Five seconds of setup, hours saved over the life of a project.
2. **Team consistency** — `<repo>/.claude/commands/` is committed; your team's shared prompts (deploy-check, security-review, standup-update) travel with the codebase.
3. **Low commitment** — skills are artifacts worth investing in. Commands are throwaways worth keeping. Write one, delete it next week if unused.

## Patterns that work

**Pre-filled scenario prompts:**

```markdown
# .claude/commands/code-review.md
Review these changes like a senior engineer. Focus on:
- Correctness first, style last
- Anything that would break under load
- Missing tests
```

**Multi-line reusable workflow:**

```markdown
# .claude/commands/ship-check.md
Before I ship this PR:
1. Run `npm test` — report results
2. Run `npm run lint` — any new violations?
3. Grep `src/` for TODO, FIXME, console.log
4. Summarize git diff against main
```

**Argument-templated:**

```markdown
---
argument-hint: "[topic]"
---

Explain $ARGUMENTS to me like I'm a smart engineer who's never touched this area.
End with: "Three follow-up questions you'd have at my level."
```

## When to promote to a skill instead

Upgrade a slash command to a skill when:

- Model-invocation helps (you want Claude to auto-suggest when relevant)
- You need tool restrictions (least privilege)
- Logic has multiple modes/flags
- Output should persist (like `/tutor` writes to docs/)

Don't upgrade just because it's getting long — a 40-line slash command is fine if it's still a single prompt.

## Debugging

**"My slash command isn't showing up."**
→ Check file is in `.claude/commands/` (exact dir) and ends in `.md`. Restart session to re-scan.

**"Wrong directory got picked up."**
→ Claude Code looks in user AND project-level dirs. Name collisions — project wins. Rename if needed.

**"Arguments aren't substituting."**
→ Use `$ARGUMENTS` verbatim. Not `${args}`, not `{{arg}}`. Check frontmatter `argument-hint` shows up when you type `/<name>`.

## Key takeaway

**Slash commands are the lowest-friction extension point.** If you've typed the same prompt twice, it's a slash command. Keep them short, name them memorably, commit project-level ones so your team shares the same toolbox. Reserve skills for the cases that actually need the extra machinery.

## See Also

- [`12-skills-in-depth.md`](12-skills-in-depth.md) — When to upgrade a command to a skill
- [`25-decision-tree.md`](25-decision-tree.md) — Slash command vs skill vs subagent vs hook

## Sources

[1] Claude Code Docs — slash commands — <https://code.claude.com/docs/en/slash-commands>
[2] Claude Code Best Practices — <https://www.anthropic.com/engineering/claude-code-best-practices>
