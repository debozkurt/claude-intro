# Chapter 05 — Auto-Memory Internals

*Last verified: 2026-04-19 — Prerequisites: Ch 02, Ch 03 — Status: Context & Memory*

**Builds on:** [`02-on-disk-model.md`](02-on-disk-model.md) · [`../agents/10-long-term-memory.md`](../agents/10-long-term-memory.md) (Long-term memory in agents).

---

## Concept

Auto-memory is Claude Code's **self-maintained learning layer** [1]. While CLAUDE.md is what *you* write to teach Claude, auto-memory is what *Claude* writes to teach itself — build commands it discovered, debugging patterns it used, your preferences as expressed through corrections. Both load at every session start. Together they're the two-half composition of persistent context [3].

On by default. Stored as plain markdown under `~/.claude/projects/<slug>/memory/`, machine-local, keyed by git repo (all worktrees of the same repo share).

## How it works

### Storage

```
~/.claude/projects/<project-slug>/memory/
├── MEMORY.md              # concise index — loaded every session
├── debugging.md           # topic file — loaded on-demand
├── build-commands.md      # topic file
└── api-conventions.md     # topic file
```

The `<project-slug>` is the absolute project path with `/` → `-`. For `~/Downloads`, the slug is `-Users-you-Downloads`. Outside a git repo, the working dir is used as the project root.

### The loading budget

**`MEMORY.md`** loads at every session start — but only the **first 200 lines or 25KB, whichever comes first** [1]. Content past the threshold doesn't load automatically; Claude reads it on demand when relevant, via the normal `Read` tool.

**Topic files** (`debugging.md`, `build-commands.md`, etc.) don't load at session start at all. Claude reads them on demand when the topic becomes relevant — typically triggered by patterns in the current conversation.

### The automatic write cycle

During a session, when Claude encounters something worth remembering — a repeated correction from you, a build command that worked after 3 tries, a package-specific quirk — it writes to auto-memory. You'll see **"Writing memory"** or **"Recalled memory"** in the TUI when this happens [1].

Writes go through `MEMORY.md` as the index: Claude adds a short one-line note plus a pointer to a topic file if it created one. Topic files accumulate depth; `MEMORY.md` stays scannable.

### Toggling it

Three ways to disable:

1. Inside session: `/memory` → toggle auto-memory off
2. Settings: `{"autoMemoryEnabled": false}` in user or local settings
3. Env: `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`

Relocate the directory by setting `autoMemoryDirectory` in user or local settings (not project settings, for security reasons [1]).

## Why it matters

Auto-memory compounds. Over weeks of use, `MEMORY.md` becomes a dense distillation of the things you keep telling Claude. The payoff: by month 3, Claude "knows" your project — build command quirks, your test-running habit, your hatred of `any` in TypeScript — without you re-teaching.

But auto-memory is **advisory**, same as CLAUDE.md. Long or stale `MEMORY.md` degrades the same way [2]. Curation discipline matters.

## Curation discipline

**Review periodically** — run `/memory` every week or two; browse what Claude wrote. Delete entries that are wrong, outdated, or too specific to last week's task.

**Keep `MEMORY.md` lean** — if it's approaching the 200-line limit, move detailed notes to topic files and shorten the index. Claude handles this automatically during sessions but occasional manual pruning helps.

**Don't treat it as a log** — auto-memory is a synthesis, not a journal. Time-stamped play-by-plays belong in commit messages and notes, not here.

**Don't trust it for critical rules** — if Claude *must* do X, that's a CLAUDE.md or hook, not auto-memory. Memory is an influence, not a guarantee.

## Debugging

**"Claude isn't following something I told it last week."**
→ Check if it made it to `MEMORY.md`. If not, the lesson didn't get captured — say it again, more clearly, and watch for the "Writing memory" indicator.

**"Claude is doing something weird and I can't find where it learned it."**
→ Grep your memory dir: `grep -r "<thing>" ~/.claude/projects/<slug>/memory/`. Auto-memory is plaintext; whatever Claude "believes" about your project is literally in those files.

**"My memory dir is massive."**
→ Topic files accumulate. Periodic cleanup is reasonable. Older project that you haven't touched in a year? `rm -rf` that slug's dir entirely.

## Key takeaway

**Auto-memory is Claude's apprentice-level journal of your project.** Loaded alongside CLAUDE.md every session, first 200 lines / 25KB hard-budgeted. Curate it like any other learned-knowledge store: review, prune, trust it directionally not strictly.

## See Also

- [`03-claude-md-discipline.md`](03-claude-md-discipline.md) — The user-authored counterpart
- [`06-context-cache.md`](06-context-cache.md) — How auto-memory affects cache hit rate
- [`../agents/10-long-term-memory.md`](../agents/10-long-term-memory.md) — General long-term memory patterns

## Sources

[1] Claude Code Docs — memory — <https://code.claude.com/docs/en/memory>
[2] Claude Code Best Practices — <https://www.anthropic.com/engineering/claude-code-best-practices>
[3] Equipping agents with Agent Skills — <https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills>
