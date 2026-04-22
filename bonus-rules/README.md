# Bonus Rules

Ready-to-install `.claude/rules/` examples that go beyond the minimal pair shipped in `lab-app/.claude/rules/`. Not part of the live demo — these are take-home artifacts for people who want richer rule patterns in their own projects.

## Install

Rules are just markdown files. Drop them into either location:

```bash
# Project-scoped (team-shared, commit to repo)
mkdir -p <your-repo>/.claude/rules
cp bonus-rules/advisor-model.md <your-repo>/.claude/rules/

# User-scoped (applies to every project, personal)
mkdir -p ~/.claude/rules
cp bonus-rules/advisor-model.md ~/.claude/rules/
```

No restart needed — rules load at the start of each session.

## What's here

- **`advisor-model.md`** — orchestration pattern for pairing a stronger "advisor" model with a cheaper "executor" sub-agent. Codifies the briefing format (Goal / Constraints / Acceptance / Start here / Out of scope), when to use vs. skip the pattern, and the common anti-patterns. Grounded in *Advisor Models: Synthesizing Instance-Specific Guidance for Steering Black-Box LLMs* (arXiv 2510.02453v2). Sourced from [ryaneggz/open-harness](https://github.com/ryaneggz/open-harness/blob/development/.claude/rules/advisor-model.md).

## How this differs from the lab-app rules

The rules in `lab-app/.claude/rules/` teach the **mechanics** of the rules system — one always-on file, one path-scoped file, both small enough to read at a glance during the demo.

The rules here teach **patterns** — reusable playbooks for how Claude should work on harder tasks. They're longer, more abstract, and most useful in real engineering codebases where sub-agent delegation actually happens.

## Adapting

Every file is plain markdown with optional YAML frontmatter. Steal freely:

- Add a `paths:` frontmatter block to scope a rule to specific files
- Trim sections that don't apply to your stack
- Rename and fork — credit is appreciated but not required
