# Chapter 09 — Session Lifecycle: Resume, Rewind, Branch

*Last verified: 2026-04-19 — Prerequisites: Ch 02 — Status: Running Claude*

**Builds on:** [`02-on-disk-model.md`](02-on-disk-model.md) · [`../agents/12-state-recovery.md`](../agents/12-state-recovery.md) (State recovery in general agents).

---

## Concept

Every Claude Code conversation is a **plaintext JSONL session file** on disk [1]. Every turn is one line of JSON. This is the mechanism that powers three otherwise-magical commands — `/resume`, `/rewind`, and `/branch` (formerly `/fork`, renamed in v2.1.77 [4]). Understanding the file-level model explains what can and can't be done.

Master-class takeaway: conversations are **durable append-only logs** you can pick up, slice, and copy. Lost context is usually a fixable problem if you know where to look.

## How it works

### Session storage

Each session lives at:

```
~/.claude/projects/<project-slug>/<session-uuid>.jsonl
```

One line per turn. Each line contains the turn's role, content, tool calls, tool results, and metadata. Completely inspectable with `cat`, `jq`, `less`.

### `/resume` (and `--resume`, `--continue`)

Pick up a previous session, preserving its full history:

| Invocation | Behavior |
|---|---|
| `claude --continue` | Resume the most recent session in this directory |
| `claude --resume` | Interactive picker across recent sessions |
| `/resume` inside a session | Same picker |

Under the hood: Claude reads the JSONL and rebuilds context. Any tool results, file reads, model reasoning from the old session is back.

### `/rewind` (Esc-Esc)

Truncate the session to an earlier turn:

- Press `Esc` twice inside the TUI
- A picker appears showing recent turns
- Select a turn; everything after it is discarded

Mechanically: the JSONL gets truncated to that line. Subsequent turns pick up from the chosen checkpoint. Useful when Claude heads in the wrong direction — rewind to before the bad turn, re-prompt differently.

### `/branch` (formerly `/fork`)

Renamed in v2.1.77 [4]. Duplicates the session up to the current point, creating a new session UUID. You can now evolve both — the original stays at its state, the branch continues under new prompts.

Use when: you want to explore two different directions from a common point. Instead of rewinding and losing the first path, branch and keep both.

## The three-tool decision

| Problem | Tool |
|---|---|
| Closed the terminal; pick up where you left off | **Resume** (`--continue` or `--resume`) |
| Wrong last turn; want to retry from a checkpoint | **Rewind** (Esc-Esc) |
| Want to explore an alternative without losing this | **Branch** (`/branch`) |

Worth noting: starting a *new* session (new `claude` invocation or new worktree) is the fourth option, and often better than rewind/branch for "totally new task" situations. Rewind when context still matters; fresh when it doesn't (Ch 06).

## Why it matters

Three unlocks:

1. **Conversations as assets, not transients** — a good 50-turn session is reusable. Resume into it weeks later; the tool calls, file reads, and reasoning are all still there.
2. **Cheap to be wrong** — rewind is your undo button. Lowers the cost of trying bold prompts.
3. **Branching exploration** — when two approaches both deserve investigation, branch lets you evolve both without coordination cost.

## Practical patterns

**"Long-lived investigation" pattern** — open a session for a bug. Over days, you add findings, tool runs, hypotheses. Resume each time. The session becomes a living debug log. When you fix it, the session is your root-cause trace.

**"Pre-flight checkpoint" pattern** — before a risky series of edits, make a throwaway safe-prompt turn ("OK, ready to proceed"). If things go wrong, rewind to that turn specifically — known-good state.

**"Two-paths" pattern** — you're weighing two architectural approaches. Explore one to completion, branch just before the critical decision, explore the other. Compare outcomes. Pick.

## Debugging

**"I lost a session."**
→ Check `~/.claude/projects/<slug>/*.jsonl` — it's probably there. Pick by timestamp.

**"Rewind lost important context."**
→ The rewind target was too early. `/resume` back to the original session JSONL — it wasn't deleted, just hidden from the current TUI.

**"Branch and original are diverging in unexpected ways."**
→ Expected — they share zero state after the branch point. If you want changes to flow, merge by prompting: paste key insights from one branch into the other.

## Key takeaway

**Sessions are files. Files are malleable.** Every expensive-feeling Claude Code conversation is recoverable, sliceable, and duplicable via `/resume`, `/rewind`, `/branch` — because JSONL on disk is what's actually powering them. Treat conversations as durable assets you can return to.

## See Also

- [`02-on-disk-model.md`](02-on-disk-model.md) — Where sessions live
- [`10-worktrees.md`](10-worktrees.md) — The git-side of parallel-path exploration
- [`../agents/12-state-recovery.md`](../agents/12-state-recovery.md) — General state-recovery patterns

## Sources

[1] Claude Code Docs — <https://code.claude.com/docs/en/>
[4] Claude Code CHANGELOG — <https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md>
