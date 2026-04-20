# Chapter 13 — Subagents in Depth

*Last verified: 2026-04-19 — Prerequisites: Ch 12 — Status: Extensions*

**Builds on:** [`../agents/13-when-to-split.md`](../agents/13-when-to-split.md) · [`../agents/14-routing-patterns.md`](../agents/14-routing-patterns.md) · [`../agents/15-merge-vs-split.md`](../agents/15-merge-vs-split.md) · [`../agents/16-shared-state.md`](../agents/16-shared-state.md).

---

## Concept

A subagent is a **delegate Claude** invoked via the parent's `Agent` tool, running in its own **fresh context window** with a **restricted tool set** [1][5]. The parent never sees the subagent's intermediate turns — only the subagent's final output. This one isolation property is the whole reason subagents exist: it keeps the parent's context clean while letting work happen that would otherwise bloat the conversation.

Claude Code's subagents are a specialization of the general multi-agent patterns in `agents/` chapters 13–16. The specific Claude Code twist: subagents are markdown files in `agents/` dir, with frontmatter (name, description, tools), plus a body that acts as the subagent's system prompt.

## How it works

### Anatomy

```
~/.claude/agents/<name>.md          # user-level
<repo>/.claude/agents/<name>.md     # project-level
```

```markdown
---
name: code-reviewer
description: Reviews diffs for safety, correctness, and style. Use when
  the user asks for a second opinion or pre-merge review.
tools: Read, Grep, Bash
---

You are a senior code reviewer.

When invoked with a diff or file range:
1. Read the changes end-to-end before commenting.
2. Flag correctness first, style last.
3. Cite file:line for every finding.
4. End with a one-line verdict: ship / fix first / blocked.
```

### Invocation

Two paths:

**Model-delegated** — the parent Claude, seeing the subagent's `description`, decides to delegate via the `Agent` tool (requires `Agent` in the parent's `allowed-tools`). The user doesn't explicitly summon the subagent.

**User-suggested** — the user asks something that clearly calls for a known subagent: "get the code-reviewer to look at this diff." Parent Claude picks up the cue and delegates.

### The isolation model

When the parent calls a subagent:

1. A new context window is allocated
2. The subagent's system prompt (body of its .md) is loaded
3. The subagent's `tools` list becomes the active tool set
4. The parent's prompt to the subagent is passed as the first message
5. The subagent runs its own loop — tool calls, reasoning, turns
6. When the subagent returns a final message, only that message bubbles back to the parent
7. The subagent's context window is discarded

The parent never sees intermediate turns. This is *the* feature.

## Why it matters

Three reasons to delegate instead of doing it inline:

1. **Parent context stays focused** — a deep research task would otherwise fill the main conversation with searches, fetches, and reading. Delegated, only the synthesis comes back.
2. **Tool restriction** — subagents get *their* tool list, not the parent's. A code-reviewer subagent can be read-only even if the parent has write access. Defense in depth.
3. **Specialization** — the subagent's body is a focused system prompt for one role. Easier to get right than a general-purpose Claude handling 10 different task types.

Not reasons to delegate:

- Simple tasks — subagents cost tokens (whole context window).
- Cooperative tasks — intermediate turns aren't visible, so you can't iterate inside the subagent's loop without re-starting.
- Random parallelism — if you just want concurrency, use worktrees (Ch 10) + parallel CLI sessions.

## The design heuristics

From `agents/13-when-to-split.md` specialized to Claude Code:

**Delegate when:** the task has a clear input → output shape, intermediate work doesn't matter, tool restriction improves safety, or the parent's context needs to stay lean for the main objective.

**Don't delegate when:** you'd want to iterate mid-task, the task needs parent's full conversation context, or the subagent would need the same tool set as the parent (no safety win).

## Worked example: `tutor-researcher`

The `/tutor` skill (Ch 12) delegates to a `tutor-researcher` subagent when `--research` is passed. Concrete shape:

- Parent `/tutor` skill has `Agent` in `allowed-tools`
- `tutor-researcher` has only read-side tools: `WebSearch, WebFetch, Read, Grep, Glob`
- Parent passes topic + context hand-off as the subagent's input prompt
- Subagent does: scope check, prior-art grep over tutor docs, external research, structured output
- Parent receives a `## Research Package: <topic>` markdown block and synthesizes the final lesson from it

The isolation is load-bearing: the parent doesn't see Claude doing 12 web searches. It sees the structured result and teaches from it.

## Debugging

**"Subagent never gets invoked."**
→ Check (a) `Agent` is in parent's `allowed-tools`, (b) subagent's `description` specifically matches the parent's prompt-matching heuristic, (c) subagent file is readable from `~/.claude/agents/` or `<repo>/.claude/agents/`.

**"Subagent output isn't what I wanted."**
→ Its system prompt (the .md body) is the whole spec. Rewrite. Especially clarify the *output format* — subagents return structured output much more reliably when the format is pinned down.

**"Subagent ran too long / used too many tokens."**
→ Its tool budget may be wrong. Restrict more. Add explicit "skim don't read in full" discipline to its body. The best-performing subagents have tight scopes and explicit stop conditions.

## Key takeaway

**Subagents trade tokens for context hygiene.** The parent gets isolation; you get tool restriction as a safety bonus. Use them for tasks with a clear input → output shape where intermediate work shouldn't pollute the parent's conversation. Don't use them as a generic "second Claude" — for true parallelism, use worktrees.

## See Also

- [`12-skills-in-depth.md`](12-skills-in-depth.md) — Skills can delegate to subagents
- [`19-subagent-orchestration.md`](19-subagent-orchestration.md) — Fan-out, routing, multi-perspective patterns
- [`25-decision-tree.md`](25-decision-tree.md) — Skill vs subagent
- [`../agents/13-when-to-split.md`](../agents/13-when-to-split.md) — General splitting heuristics
- [`../agents/14-routing-patterns.md`](../agents/14-routing-patterns.md) — Routing across subagents

## Sources

[1] Claude Code Docs — <https://code.claude.com/docs/en/>
[5] Agent SDK subagents — <https://platform.claude.com/docs/en/agent-sdk/subagents>
