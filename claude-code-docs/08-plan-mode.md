# Chapter 08 — Plan Mode

*Last verified: 2026-04-19 — Prerequisites: Ch 07 — Status: Running Claude*

**Builds on:** [`../agents/18-human-in-the-loop.md`](../agents/18-human-in-the-loop.md) (approval gates).

---

## Concept

Plan mode is a **read-only execution state** where Claude investigates, reasons, and proposes — but never edits, runs state-changing shell commands, or calls state-changing MCP tools [1][2]. The output is a structured plan that you approve (execute), reject (abandon), or iterate on. It's the "dry run" primitive, and it's the master-class move for any change that's bigger than a one-liner.

The Anthropic-internal workflow Anthropic itself documents as their preferred default: **"Explore, plan, code, commit"** [2]. Plan mode is the second step made concrete.

## How it works

### Entering

Three ways:

1. `Shift+Tab` until the TUI shows "Plan mode"
2. Start the session with `claude --permission-mode plan`
3. From an ongoing turn, switch via the mode indicator

### What Claude does in plan mode

- Reads files, runs `Grep`/`Glob`, searches the web — all fine
- Reasons through the problem
- Writes a **structured plan**: files to touch, order, risks, open questions
- Stops there

What Claude *won't* do:

- Call `Edit`, `Write`
- Run `Bash` commands that change state (destructive Bash may still be blocked by deny even in plan)
- Invoke state-changing MCP tools
- Use `Agent` to delegate work (depending on harness version — verify in your CHANGELOG [4])

### Approving

At plan's end, you get three options [1]:

- **Execute** — switches out of plan mode and executes the plan
- **Reject** — discard the plan, go back to normal
- **Iterate** — ask Claude to revise the plan before deciding

## Why it matters

Three wins that compound:

1. **Catches misunderstandings early** — a misread of your intent becomes a visible plan, not 200 lines of wrong code.
2. **Forces Claude to think before acting** — reasoning that would otherwise happen mid-edit now happens up front, with your eyes on it.
3. **Creates a reviewable artifact** — the plan itself is useful: you can copy it into a PR description, a design doc, a teammate's Slack [2].

The #1 intermediate-level mistake is skipping plan mode for "quick" changes that turn out not to be quick. The #1 senior-level habit is starting in plan for anything touching more than one file.

## When to reach for it

- **Non-trivial refactors** — anything touching 3+ files
- **Unfamiliar codebases** — when you don't know the blast radius yet
- **Ambiguous requests** — force-surface the ambiguity before it becomes bad code
- **High-stakes surfaces** — auth, payments, data migration, production config
- **Teaching moments** — a good plan from Claude explains the problem in a way worth reading

When *not* to reach for it:

- Trivial edits you'd type yourself faster
- Exploratory conversation ("what are my options for X?")
- Debugging where you need to run things to see what happens

## Getting better plans

**Ask for structure** — plans are more useful when they name files, dependencies, risks:

> "Draft a plan. For each file, name the change. List risks. Flag open questions before executing."

**Ask for tradeoffs** — for design-level work:

> "Plan two approaches — a minimal fix and a proper refactor. Compare before recommending."

**Iterate before executing** — "what if X?" "what about Y?" is cheap in plan mode; expensive after execution.

## Debugging

**"Claude started editing in plan mode."**
→ Check the status bar — you may have switched out by accident. `Shift+Tab` back into plan.

**"Plan is too shallow."**
→ Ask for more: "Plan is too abstract. For each file, name the specific change and why." Iteration in plan is cheap.

**"Plan looks right but execution went wrong."**
→ Rewind (`Esc`-`Esc`) to just after the plan was approved, then ask Claude to re-read the plan and check each step before acting.

## Key takeaway

**Plan mode is the senior engineer's default for anything non-trivial.** The cost of a five-minute plan review is always less than the cost of unwinding a bad edit. If the plan surfaces a misunderstanding, you saved an hour. If it confirms your intent, you saved zero minutes and moved forward with confidence.

## See Also

- [`07-permission-modes.md`](07-permission-modes.md) — Plan mode as one of four permission modes
- [`09-session-lifecycle.md`](09-session-lifecycle.md) — Rewind to before plan execution
- [`25-decision-tree.md`](25-decision-tree.md) — Plan mode vs. "just do it" decision

## Sources

[1] Claude Code Docs — <https://code.claude.com/docs/en/>
[2] Claude Code Best Practices — <https://www.anthropic.com/engineering/claude-code-best-practices>
[4] Claude Code CHANGELOG — <https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md>
