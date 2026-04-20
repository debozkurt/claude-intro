# Chapter 26 — Claude Code Patterns (2026 Update)

*Last verified: 2026-04-19 — Prerequisites: most of this guide — Status: Meta*

**Builds on:** [`../agents/29-modern-patterns.md`](../agents/29-modern-patterns.md) — this chapter is the Claude-Code-specific 2026 supplement to that chapter's broader agent-patterns view.

---

## Concept

Chapter 29 of the `agents/` series (written 2025) laid out the general pattern vocabulary: agent harnesses, Ralph loops, multi-layer memory, subagent fan-out, MCP, prompts-as-code. This chapter does the **2026 update specific to Claude Code** — what's changed, what's been renamed, what's now convention, and what debates are still live.

Practical framing: if you've internalized the general patterns, this chapter is your diff against current Claude Code reality.

## What's changed since Ch 29

### Renames

- **`/fork` → `/branch`** (v2.1.77) — conversation forking is now `/branch` [4]. Old docs and blogs still mention `/fork`; same feature.
- **"Claude Code SDK" → "Agent SDK"** (late 2025) — the library was rebranded to reflect its general-purpose agent runtime role. Same harness; broader positioning.
- **`.claude/rules/` path-scoping added** (2025+) — the `paths:` YAML frontmatter for file-scoped rules is newer than Ch 29 and deserves its own chapter (Ch 04) here.

### New / newly-formalized primitives

- **Auto memory** — what Ch 29 described abstractly as "Claude writing its own notes" is now the concrete `~/.claude/projects/<slug>/memory/MEMORY.md` feature with a 200-line / 25KB budget (Ch 05).
- **Plan mode as a permission mode** — plan isn't just a prompt technique anymore; it's a first-class permission state (Ch 07, Ch 08).
- **Subagents as first-class files** — Ch 29 described specialist agents; Claude Code now formalizes them as `.md` files in `agents/` with `tools:` frontmatter (Ch 13).

### Cache TTL (April 2026, ongoing debate)

Prompt cache TTL default dropped to 5 minutes in early April 2026 [4]. Practitioner community reaction is mixed — fast workflows unaffected or cheaper; slow-paced conversational use materially more expensive. Anthropic's stated rationale: net-cheaper for one-shot and short-turn patterns. The TTL flag for keeping longer cache is available in the SDK and some CLI configurations; check the CHANGELOG [4] for current state.

Expect this to shift again. Mark your chapter's "Last verified" date and re-check at quarterly cadence.

## New patterns the community has crystallized

### The "worktree + YOLO" pattern

Ch 29 mentioned YOLO mode briefly; the 2026 practitioner consensus [8] is specifically:

- Bypass mode *only* inside a throwaway git worktree
- Tests as guardrails (failing tests block merge-back, so bypass can't silently break the main branch)
- Short-lived — 15-minute "get it done" sessions, not persistent

This is the pattern that reconciles Anthropic-official safety guidance with what top practitioners actually do. Both camps agree: *inside a sandbox*, bypass is fine; *outside*, it's not.

### The "skill + researcher subagent" pattern

Formalized in this guide's own `/tutor` + `tutor-researcher` example. General shape:

- Main skill owns the procedure and the output
- Research subagent owns the ingredient-gathering in isolation
- Parent synthesizes ingredients into the final output, with citations

Generalizes beyond tutor — any skill where external research adds value (RFC-grounded SIP lessons, security audits, framework comparison) can follow the same shape. Expect this to become a standard pattern in the skill marketplace.

### The "hook for observability" pattern

Ch 29 mentioned hooks briefly. 2026 practice [8] specifically uses hooks as:

- Structured-log emitters for every tool call (audit trail)
- Slack/desktop notifiers for long-running skill completion
- Deterministic guardrails that no model can bypass (hard `deny` logic with context)
- OTel span emitters in production deployments

Hooks are less "optional extension" and more "production baseline" at this point.

### The "MCP as portability layer" pattern

2025 saw the MCP ecosystem mature — registry launch, OpenAI + Google DeepMind adoption [7][10]. 2026 practitioner guidance has crystallized: *any tool integration you might want to use from more than one agent client should be an MCP server*. Skills are Claude Code-specific; MCP servers are portable.

## Ongoing debates (2026)

### CLAUDE.md length

Short (Anthropic-official, under 200 lines) vs. comprehensive (some teams commit 500+ line CLAUDE.md with deep convention documentation). Both defensible. No resolution.

### Skills vs subagents for specialization

Official docs [3] say both work. Practitioner drift [8] toward "skills for knowledge/procedure, subagents for tool-restricted isolated execution." Still no canonical rule.

### YOLO in CI

Specifically for short-lived CI runs in ephemeral containers with no persistent secrets — community is split. Some shops bypass permissions in CI because the ephemeral container *is* the sandbox. Others keep `settings.json` tight and never bypass. Active topic.

### Auto-memory curation cadence

Some practitioners aggressively prune `~/.claude/projects/<slug>/memory/`; others treat it as pure additive log. No consensus. Likely depends on project lifespan — short projects tolerate pure-additive; long projects benefit from curation.

## What's probably coming next

Flag these as "watch list" — likely to evolve in the next 6-12 months:

- **More sophisticated hook event types** — richer context per event, perhaps structured JSON inputs rather than env vars
- **Skill composition** — formal skill-calls-skill (today informal, achievable via prompts)
- **MCP registry quality marks** — as the registry matures, Anthropic or the spec owners likely introduce some quality/verification signal
- **Persistent agent sessions** — long-running Ralph loops may get first-class CLI/SDK support vs. the current script-and-loop approach
- **Cost transparency** — better `/cost` features, possibly per-skill cost attribution

## Key takeaway

**Claude Code in 2026 is more primitive-rich, more production-ready, and still evolving fast.** The patterns in Ch 29 still hold; the specifics of naming, default cache behavior, and recommended practices have moved. Master-class users maintain currency — re-read official docs quarterly, track CHANGELOG monthly, and mark every doc with a "Last verified" date that honestly reflects when you checked. This chapter dated 2026-04-19 is itself a snapshot.

## See Also

- [`../agents/29-modern-patterns.md`](../agents/29-modern-patterns.md) — The general-agent-pattern chapter this specializes
- [`appendix-changelog.md`](appendix-changelog.md) — The rename / feature-change log for this guide
- [`27-recipes.md`](27-recipes.md) — Concrete 2026 workflows

## Sources

[3] Equipping agents with Agent Skills — <https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills>
[4] Claude Code CHANGELOG — <https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md>
[7] Model Context Protocol — <https://modelcontextprotocol.io>
[8] Shankar — "How I Use Every Claude Code Feature" — <https://blog.sshh.io/p/how-i-use-every-claude-code-feature>
[10] Claude Code MCP docs — <https://code.claude.com/docs/en/mcp>
