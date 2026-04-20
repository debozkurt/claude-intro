# Chapter 27 — Recipes: 10 Workflows to Steal

*Last verified: 2026-04-19 — Prerequisites: Ch 11–14, Ch 17–18 — Status: Meta*

**Fresh ground** — concrete workflows that demonstrate the decision tree (Ch 25) in action.

---

## Concept

Ten recipes. Each maps a concrete problem to the right primitive (or composition of primitives), with a minimal working example. Steal and adapt; these are starting points, not prescriptions.

---

## 1 — Pre-commit linter as a hook

**Problem:** every `Edit` should trigger ESLint on the affected files.

**Primitive:** Hook (`PreToolUse` won't work — the file isn't edited yet; use `PostToolUse`).

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit",
      "hooks": [{
        "type": "command",
        "command": "for f in $CLAUDE_FILE_PATHS; do case \"$f\" in *.ts|*.tsx|*.js) npx eslint --fix \"$f\" ;; esac; done"
      }]
    }]
  }
}
```

Deterministic, no model involvement, runs every edit. Ch 14.

---

## 2 — Safe YOLO experiment sandbox

**Problem:** run an exploratory task fast without permission prompts, but safely.

**Primitive:** Worktree + bypass mode.

```bash
git worktree add ../yolo-$(date +%s) experiment/yolo
cd ../yolo-*
claude --dangerously-skip-permissions
# ...experiment...
# when done:
cd -
git worktree remove ../yolo-*
git branch -D experiment/yolo
```

Ch 07 + Ch 10. Wrap as a shell function for reuse.

---

## 3 — Nightly code-health audit

**Problem:** once a day, audit your repo for TODOs, stale branches, test coverage, security notes.

**Primitive:** Scheduled headless + skill.

```bash
# ~/bin/daily-audit.sh
cd ~/projects/my-app
claude -p "$(cat ~/prompts/audit.md)" > /tmp/audit-$(date +%F).md
```

Cron: `0 6 * * * ~/bin/daily-audit.sh`. Ch 17.

The prompt file can be a skill invocation: `/audit-repo` with accumulated rules.

---

## 4 — PR security review as a subagent

**Problem:** every PR should get an automated security-focused review.

**Primitive:** Subagent + headless in CI.

```yaml
# .github/workflows/security-review.yml
- run: |
    git diff origin/main..HEAD > /tmp/diff.patch
    claude -p "Use the security-reviewer subagent to review /tmp/diff.patch. \
               Exit 1 if any critical finding." \
      --permission-mode acceptEdits
```

`.claude/agents/security-reviewer.md` contains the role definition. Ch 13 + Ch 17.

---

## 5 — MCP-powered internal tool

**Problem:** Claude needs to query your company's internal feature-flag system.

**Primitive:** MCP server.

Build a small MCP server (stdio, ~100 lines of Python) that exposes:
- `list_flags()` — returns active flags
- `get_flag(name)` — returns flag config
- `get_flag_history(name)` — audit trail

Commit `.mcp.json` with the server registration. Teammates clone; `/mcp` lists it; Claude uses it naturally. Ch 16.

---

## 6 — Skill + subagent research pipeline

**Problem:** teach a concept rigorously, grounded in current sources.

**Primitive:** Skill + subagent composition.

- Skill: `/tutor` — handles the teaching format, persistence, 5-layer framework
- Subagent: `tutor-researcher` — gathers sources + prior art
- Hook: `PostToolUse` on `Write` — notifies when lesson is persisted

Full implementation in the `bonus-skills/` directory of the meetup repo. Ch 12, Ch 13, Ch 14, Ch 25.

---

## 7 — Long-running doc ingestion

**Problem:** extract structured facts from 1000 PDFs in a research folder.

**Primitive:** Ralph loop via SDK.

```python
from claude_agent_sdk import run_agent
import sqlite3, glob

db = sqlite3.connect("findings.db")
for pdf in glob.glob("corpus/*.pdf"):
    if db.execute("SELECT 1 FROM findings WHERE pdf = ?", (pdf,)).fetchone():
        continue  # idempotent
    result = run_agent(
        prompt=f"Extract key claims from {pdf}. Return JSON.",
        allowed_tools=["Read"],
        model="haiku",
    )
    db.execute("INSERT INTO findings VALUES (?, ?)", (pdf, result.final_message))
    db.commit()
```

Idempotent, restart-safe, cheap (Haiku). Ch 20 + Ch 22.

---

## 8 — `/onboard-repo` skill

**Problem:** when you join a new codebase, generate a personal onboarding doc.

**Primitive:** Skill.

```markdown
---
name: onboard-repo
description: Generate a personal onboarding note for this repo — entry points,
  key abstractions, dev loop, pitfalls. Use when starting on a new codebase.
user-invocable: true
allowed-tools: Read, Grep, Glob, Bash(git log:*), Bash(ls:*)
---

Produce a 1-page onboarding doc:
1. What does this project do? (read README, package.json / pyproject)
2. Entry points — where does execution begin?
3. Core abstractions — the 3 most important concepts to understand
4. Dev loop — how to run, test, debug
5. Gotchas — check git log for recent hotfixes; flag any "tricky" areas

Write the result to `~/notes/onboarding-<repo-name>.md`.
```

Ch 12.

---

## 9 — Cost-tracked session via hook

**Problem:** get per-session cost summaries in a dashboard.

**Primitive:** Hook + external store.

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "echo \"{\\\"ts\\\":\\\"$(date -u +%FT%TZ)\\\",\\\"session\\\":\\\"$CLAUDE_SESSION_ID\\\",\\\"cost\\\":\\\"$CLAUDE_SESSION_COST\\\"}\" >> ~/.claude-cost.jsonl"
      }]
    }]
  }
}
```

Ship to Loki / Grafana. Ch 14 + Ch 23.

---

## 10 — Team-shared "deploy-check" slash command

**Problem:** before deploy, everyone should run the same pre-deploy checklist.

**Primitive:** Committed slash command.

```markdown
<!-- .claude/commands/deploy-check.md -->
---
description: Pre-deploy checklist — run before every deploy
---

Verify before shipping:
1. All tests pass: `npm test`
2. No secrets in the diff: `git diff origin/main | grep -iE 'key|token|password'` (should be empty)
3. Changelog updated with this version
4. No unresolved TODO-FIXME-in-diff
5. Run `npm audit --audit-level high` — any critical?

Report PASS / FAIL per check. Halt on first FAIL.
```

Commit to `.claude/commands/`; teammates get it automatically. Ch 11.

---

## How to pick your first recipe

Start with the one that's annoying you most right now. Hooks (1, 9) are zero-risk to try — they just observe. Slash commands (10) are lowest-friction to build. Skills (6, 8) are where you commit. Subagents (4, 6) are for when context hygiene matters. Recipes 2, 5, 7 are power-user territory.

## Key takeaway

**The productivity gain from Claude Code at mastery level comes from composing the primitives you've already learned.** These ten recipes are templates — most mature Claude Code users have a personal library of 10-20 customized versions of these. Steal freely, adapt to your actual workflows, and share the good ones with your team.

## See Also

- [`25-decision-tree.md`](25-decision-tree.md) — Why each recipe picked its primitive
- All extension-primitive chapters (11-14)

## Sources

[1] Claude Code Docs — <https://code.claude.com/docs/en/>
[2] Claude Code Best Practices — <https://www.anthropic.com/engineering/claude-code-best-practices>
[8] Shankar — "How I Use Every Claude Code Feature" — <https://blog.sshh.io/p/how-i-use-every-claude-code-feature>
[9] Long-running Claude for scientific computing — <https://www.anthropic.com/research/long-running-Claude>
