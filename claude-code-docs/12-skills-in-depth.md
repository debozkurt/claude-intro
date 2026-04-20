# Chapter 12 — Skills in Depth

*Last verified: 2026-04-19 — Prerequisites: Ch 01 (mental model), Ch 03 (CLAUDE.md discipline) — Status: Extensions*

**Builds on:** [`../agents/03-tools.md`](../agents/03-tools.md) (Tools) · [`../agents/07-prompts-as-code.md`](../agents/07-prompts-as-code.md) (Prompts as Code).

---

## Concept

A **skill** is a markdown file with YAML frontmatter that becomes callable — by the user as a slash command, or automatically by the model when the user's prompt matches its `description` [3][11]. The body of the skill is appended to the system prompt when invoked, turning the skill into a **procedural artifact**: a prompt + tool budget + expected behavior, version-controlled, testable, shareable.

The key design insight from Anthropic's skills paper [3]: skills use **progressive disclosure**. The frontmatter is always loaded so Claude can decide whether to invoke; the body is only loaded when it does. This is how you can have dozens of skills installed without blowing your context budget — they're cheap until used.

## How it works

### Anatomy

```markdown
---
name: <slug>              # the /slash-command name
description: <one-line>   # used for auto-invocation matching
user-invocable: true      # shows up in /slash menu
allowed-tools: <list>     # tight tool budget for least privilege
argument-hint: "[args]"   # placeholder in UI
---

# Skill body (markdown)

Instructions to Claude when invoked: steps, rules, examples, constraints.
The body IS the system prompt the model sees when this skill is active.
```

Location:

- `~/.claude/skills/<name>/SKILL.md` — personal, all projects
- `<repo>/.claude/skills/<name>/SKILL.md` — team-shared, committed

### Two invocation paths

**User-invoked** — the user types `/<name>` (optionally with args). Works if `user-invocable: true`.

**Model-invoked** — the user's prompt matches the skill's `description` strongly enough that Claude auto-suggests invoking it. The model sees all installed skills' frontmatter at session start and decides when to reach [3].

Both paths result in the skill's body being appended to the active prompt and the skill's `allowed-tools` becoming the active tool set.

### Progressive disclosure explained

Without skills, every procedure you wanted Claude to know would have to live in CLAUDE.md — paying token cost every turn. With skills:

| Phase | What's loaded |
|---|---|
| Session start | Every skill's frontmatter (small — ~5 lines each) |
| User prompt | Frontmatter only — model decides whether to invoke |
| Skill invoked | That skill's body now in context |
| After skill turn | Body can be garbage-collected by `/compact` |

This is why you can install 50 skills without blowing your context budget. The body — where the real content lives — only loads when needed [3][11].

## The critical skill: writing descriptions that trigger reliably

The #1 failure mode for model-invoked skills is a bad `description`. Claude auto-invokes when a user's prompt *matches* the description. Bad descriptions either over-trigger (the skill runs when you didn't want it) or under-trigger (the skill never fires even when relevant).

**Bad description:** `"Helpful skill for coding."` — too vague; will never match anything specifically.

**Bad description:** `"A tool."` — meaningless.

**Good description:** `"Principal engineer mentor that explains concepts with depth, theory, and best practices. Use when the user says 'tutor' or wants to understand the 'why' behind code."`

Three rules for descriptions [11]:

1. **Name specific trigger words** — "use when the user says 'X'" or "mentions 'Y'"
2. **Be concrete about what it does** — not "helps with code" but "reviews diffs for safety and correctness"
3. **Name the negative space** — "don't invoke for simple syntax questions" if overtriggering is a concern

Validate by having Claude read your skill's description and ask: "Given this description, when should I invoke this skill?" If the answer is vague, rewrite.

## Tool-budget discipline

`allowed-tools` is a **security and correctness boundary**. A skill shouldn't have access to tools it doesn't need. Common patterns:

| Skill type | `allowed-tools` |
|---|---|
| Read-only analysis | `Read, Grep, Glob` |
| Writes docs | `+ Write, Edit` |
| Runs tests | `+ Bash(npm test:*)` — Bash glob-scoped |
| Delegates | `+ Agent` |
| Web research | `+ WebSearch, WebFetch` |

Two design principles:

- **Least privilege** — if the skill doesn't need `Bash`, don't grant it
- **Scoped Bash** — if it does need Bash, scope with `Bash(<pattern>:*)` rather than raw `Bash`

`Bash(rm -rf*)` should almost never be in any skill's allowlist. When it is, there should be a clear narrative for why — and probably a `deny` rule in `settings.json` as backup (Ch 24).

## Worked example: `/tutor`

The `/tutor` skill (`~/.claude/skills/tutor/SKILL.md`, 285+ lines) is a real, structured example:

```yaml
---
name: tutor
description: Principal engineer mentor that explains concepts with depth,
  theory, and best practices. Context-aware teaching with persistent lesson
  docs. Use when the user says "tutor" or wants to understand the "why"
  behind code.
user-invocable: true
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, WebSearch, Agent
argument-hint: "[topic] [--socratic] [--deep] [--research] [--debug] [--progress]"
---
```

Design choices worth stealing:

1. **Flags compose** — `/tutor --socratic --debug "X"` is valid. Users memorize one skill + flags, not eight skills.
2. **Body enforces a framework** — the 5-layer teaching structure (Situation → Mechanism → Principle → Adjacent → Pattern Recognition) is in the body. Every lesson follows it.
3. **Persistence pattern** — the skill writes to `~/.claude/skills/tutor/docs/<domain>.md`. Output accumulates across sessions. The skill builds a personal knowledgebase.
4. **`Agent` in allowed-tools** — enables delegation to the `tutor-researcher` subagent when `--research` is passed (see Ch 13 + `tutor-researcher.md`).

## When skill vs. when something else

Full decision tree in Chapter 25. Short preview:

- **Skill** — procedural knowledge, invoked on demand, survives across sessions
- **Slash command** (Ch 11) — a prompt template, user-invoked only, no fancy logic
- **Subagent** (Ch 13) — isolated execution with its own context window and tool restrictions
- **Hook** (Ch 14) — runs deterministically on an event, model not involved

If the answer is "I want Claude to know how to do X whenever I ask" — skill. If "I want a specific prompt I can type fast" — slash command. If "I want to delegate research/review so my main context stays clean" — subagent. If "every time X happens, run this" — hook.

## Key takeaway

**A skill is a versioned, described, tool-budgeted prompt.** Progressive disclosure means skills are nearly free until used. The hardest craft is writing the `description` — specific enough that Claude knows when to invoke, restrictive enough not to over-fire. Once the description works, the body is straightforward: a structured system prompt that enforces how you want the task done.

## See Also

- [`11-slash-commands.md`](11-slash-commands.md) — The simpler, user-only primitive
- [`13-subagents-in-depth.md`](13-subagents-in-depth.md) — When isolation beats inline
- [`25-decision-tree.md`](25-decision-tree.md) — Which primitive for which problem
- [`../agents/03-tools.md`](../agents/03-tools.md) — Tools as agent building blocks
- [`../agents/07-prompts-as-code.md`](../agents/07-prompts-as-code.md) — Prompts-as-code framing

## Sources

[3] Equipping agents for the real world with Agent Skills — <https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills>
[11] Skill authoring best practices — <https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices>
