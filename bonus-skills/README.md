# Bonus Skills

Ready-to-install skills for Claude Code. These are the take-home artifacts from the panel.

## Install

```bash
# /tutor skill
mkdir -p ~/.claude/skills/tutor
cp bonus-skills/tutor/SKILL.md ~/.claude/skills/tutor/SKILL.md
mkdir -p ~/.claude/skills/tutor/docs   # where lessons persist

# tutor-researcher subagent
mkdir -p ~/.claude/agents
cp bonus-skills/tutor-researcher/tutor-researcher.md ~/.claude/agents/
```

Restart your Claude session. Invoke:

- `/tutor "topic"` — standard lesson
- `/tutor --research "topic"` — delegates to the subagent first; lesson includes citations and a Sources section
- `/tutor --progress` — see what you've learned across all domains

## What's here

- **`tutor/`** — the skill we walked through live. A principal-engineer mentor that delivers layered lessons (Situation → Mechanism → Principle → Adjacent → Pattern Recognition), supports Socratic / Debug / Research modes, and persists lessons to `~/.claude/skills/tutor/docs/<domain>.md` as a growing personal knowledge base.
- **`tutor-researcher/`** — the companion subagent. When `/tutor --research` is invoked, delegates here first. Gathers 3-6 authoritative sources, cross-references prior tutor lessons, and returns structured findings so `/tutor` can deliver a citation-grounded lesson with a Sources bibliography. Install to `~/.claude/agents/tutor-researcher.md`.

## Adapting skills

Every file in this directory is a single markdown file with YAML frontmatter — steal freely:

- Change `name:` to rename the slash command
- Tighten or loosen `allowed-tools:`
- Swap the body's framework for your own
- Move to `<repo>/.claude/skills/` to make it team-shared instead of personal
