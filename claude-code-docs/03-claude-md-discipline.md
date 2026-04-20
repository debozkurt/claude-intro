# Chapter 03 — CLAUDE.md Discipline

*Last verified: 2026-04-19 — Prerequisites: Chapter 01 (mental model) — Status: Context & Memory*

**Builds on:** [`../agents/29-modern-patterns.md` § Multi-scope instructions](../agents/29-modern-patterns.md) · [`../agents/09-context-and-cache-engineering.md`](../agents/09-context-and-cache-engineering.md) (Context & cache engineering).

---

## Concept

`CLAUDE.md` is **advisory context**, not enforced configuration [2]. Claude treats it as high-weight guidance, but a model can and will ignore it — especially when the file is long, inconsistent, or unfocused. Mastering Claude Code means mastering `CLAUDE.md` discipline: *what* goes in, *where* in the hierarchy, *how long*, and *how to prune*.

The single biggest intermediate-level skill gap is writing disciplined CLAUDE.md files. Beginners write too little ("I gave it my whole README"); journeymen write too much ("500 lines of everything I might ever want Claude to know"). Neither works well. The craft is **short, pruned, layered**.

## How it works

### The hierarchy

At session start, Claude walks up from your working directory and loads every `CLAUDE.md` it finds [1][2]. All are concatenated, with *more specific loaded later* (so more specific wins on conflicts):

```
~/.claude/CLAUDE.md              ← user-level preferences, all projects
<repo>/CLAUDE.md                 ← project-level, committed
<repo>/.claude/CLAUDE.md         ← alternate project location, same scope
<repo>/some/subfolder/CLAUDE.md  ← loaded on-demand when Claude touches that dir
```

Subfolder CLAUDE.md files are loaded *lazily* — only when Claude reads files in that subdirectory. This is load-bearing for monorepos: each team's CLAUDE.md lives with their code, doesn't pollute the global context.

### The size budget

Every token in CLAUDE.md is a token in every turn's prompt. Size ≠ quality [2][6]:

| CLAUDE.md size | Typical adherence | Symptom of over-size |
|---|---|---|
| < 100 lines | ~95% | Miss rare cases |
| 100–200 lines | ~85% | Good balance |
| 200–500 lines | ~70% | Specific rules "get lost" in noise |
| > 500 lines | < 60% | Model starts doing its own thing |

These are practitioner observations, not guaranteed numbers [8]. The mechanism is mundane: the model weighs context globally, and more tokens competing for attention means each individual rule has less pull.

### The three categories of content

Healthy CLAUDE.md files separate three kinds of content into three homes:

| Category | Belongs in | Why |
|---|---|---|
| **Advisory context** ("how we think about this project") | `CLAUDE.md` | Needs to be top-of-mind every turn |
| **Enforceable rules** ("always do X, never do Y") | `.claude/hooks/` as `PreToolUse` blocks | Deterministic, not advisory (Ch 14) |
| **Invokable procedures** ("when asked to do X, follow these steps") | `.claude/skills/<name>/SKILL.md` | On-demand, doesn't cost context every turn (Ch 12) |

Most "CLAUDE.md is too long" problems resolve by asking **"which category is this rule?"** and moving it accordingly.

### Imports

`@path` syntax pulls in external files at session start, expanded inline with up to 5 hops of recursion [1]:

```markdown
# Project

## Architecture
See @docs/architecture.md for system context.

## Style
Follow @.claude/STYLE.md — enforced in CI.
```

This is how you keep CLAUDE.md as a *table of contents* to existing docs rather than re-documenting your repo. The imported files cost tokens just as if pasted inline — this is reorganization, not compression.

## Why it matters

CLAUDE.md is where the cost / adherence / team-sharing tradeoffs collide:

- **Too short** — Claude doesn't know your conventions, makes the same mistakes a new hire would
- **Too long** — adherence decays, *and* every turn pays the full token cost [6]
- **Too user-specific, committed** — your personal preferences leak into teammates' sessions
- **Too team-specific, user-level** — teammates can't reproduce your working environment

The discipline pays rent three ways: better model behavior, lower per-turn token cost (and therefore better cache hit rates — Ch 06), and better team ergonomics.

## Common failure modes and fixes

**"My CLAUDE.md is 800 lines and Claude is ignoring half of it."**
→ Audit by category (advisory / enforceable / invokable). Move enforceables into hooks, invokables into skills. What remains should be < 200 lines. Use `.claude/rules/` with `paths:` frontmatter for anything that only matters in specific dirs (Ch 04).

**"Claude followed my CLAUDE.md at the start of the session but drifted by the end."**
→ Long conversation pushed CLAUDE.md far from the recent turns in effective attention. Invoke `/compact` to recenter, or consider moving critical rules into a skill whose body re-anchors them on invocation.

**"My team's CLAUDE.md conflicts with my personal preferences."**
→ Use the hierarchy as designed. Team conventions in `<repo>/CLAUDE.md` (committed). Your personal preferences in `~/.claude/CLAUDE.md`. Both get loaded; no conflict, no coordination problem.

**"I don't know what's actually loading."**
→ Run `/memory` — lists every loaded CLAUDE.md and rules file in the current session. Also audits auto-memory (Ch 05).

## Key takeaway

**CLAUDE.md is the file you write down what you'd otherwise re-explain.** Keep it short. Prune ruthlessly. Anything enforceable belongs in hooks; anything procedural belongs in skills; anything project-wide belongs in the repo CLAUDE.md; anything personal belongs in `~/.claude/CLAUDE.md`. Use the hierarchy instead of cramming everything into one file.

## See Also

- [`01-mental-model.md`](01-mental-model.md) — Why CLAUDE.md is part of context assembly every turn
- [`04-claude-rules.md`](04-claude-rules.md) — The path-scoped alternative for bigger projects
- [`06-context-cache.md`](06-context-cache.md) — How CLAUDE.md size interacts with prompt caching
- [`../agents/29-modern-patterns.md`](../agents/29-modern-patterns.md) — Multi-scope instructions in general agent design

## Sources

[1] Claude Code Docs — memory — <https://code.claude.com/docs/en/memory>
[2] Claude Code Best Practices — <https://www.anthropic.com/engineering/claude-code-best-practices>
[6] Effective Context Engineering for AI Agents — <https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents>
[8] Shrivu Shankar, "How I Use Every Claude Code Feature" — <https://blog.sshh.io/p/how-i-use-every-claude-code-feature>
