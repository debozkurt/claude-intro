# Chapter 06 — Context and Cache Engineering

*Last verified: 2026-04-19 — Prerequisites: Ch 03, Ch 05 — Status: Context & Memory*

**Builds on:** [`../agents/09-context-and-cache-engineering.md`](../agents/09-context-and-cache-engineering.md) (Context & cache engineering) — this chapter specializes that general framework to Claude Code's specific cache model and controls.

---

## Concept

Two independent but entangled systems shape every turn: the **context window** (what the model sees) and the **prompt cache** (what's billed as cache-read vs cache-write). Mastering Claude Code at scale means tuning both — not as separate concerns but as one system where layout decisions affect both simultaneously [6].

The April 2026 change that rippled through practitioner workflows: prompt cache TTL default dropped to **5 minutes** for many Claude Code requests. Long idle gaps between turns now re-pay the cache write. Mid-turn cache hits are still cheap; slow, human-paced conversations now pay more than they did in 2025 [4].

## How it works

### What gets cached

Claude Code structures each turn's prompt as:

```
[stable prefix: system + CLAUDE.md layers + rules + skill frontmatter]
[semi-stable middle: conversation history]
[volatile suffix: current turn + tool results]
```

The **stable prefix** is what caches well. Keep it unchanged across turns and subsequent cache reads cost a fraction of a fresh write [6]. Changing CLAUDE.md mid-session invalidates the cache — every subsequent turn pays the write cost again until the new prefix settles.

### The TTL

Default TTL dropped to 5 minutes in April 2026 [4]. Practical implication:

| Inter-turn gap | Cache behavior |
|---|---|
| < 5 min | Warm cache hit, cheap |
| 5–30 min | Cache expired, new write |
| > 30 min | Likely new session worth considering (`/resume` cost may exceed fresh) |

Some higher-traffic users kept the older 1-hour default via explicit flags or SDK usage. Check [4] for latest.

### Active controls

| Command | Effect on context | Effect on cache |
|---|---|---|
| `/compact` | Summarizes older turns | Invalidates — cache writes on next turn |
| `/clear` | Wipes conversation | Invalidates but rebuilds from known-stable prefix |
| `/resume` | Loads a session JSONL | Cache warms on first real turn |
| `/rewind` | Truncates to an earlier turn | Invalidates from that point forward |
| Editing CLAUDE.md | Changes the stable prefix | Invalidates entire cache |

## Why it matters

Three practitioner wins from cache discipline:

1. **Cost** — a 60-turn session with well-cached prefix is 3–5× cheaper than one with mid-session CLAUDE.md edits [6][8].
2. **Latency** — cache reads return faster than cache writes, so cached sessions feel snappier.
3. **Adherence** — shorter, more disciplined CLAUDE.md both caches better and produces better model behavior (Ch 03 — same root cause).

## The compact / clear / fresh decision

Every few dozen turns the session gets heavy. Three options:

**`/compact`** — keep this session alive, compress older turns. Good when: recent turns matter; starting over would lose context.

**`/clear`** — wipe conversation, keep CLAUDE.md. Good when: switching tasks entirely; older turns are irrelevant.

**Fresh session (new `claude` or `/branch`)** — start clean. Good when: you want maximum cache warmth on the stable prefix and don't need any conversation history. Also: a new worktree + fresh session is how you "branch" an exploration (Ch 10).

Don't let sessions grow indefinitely. A stale 200-turn session costs more and performs worse than a 20-turn one with the same CLAUDE.md.

## Cache-friendly CLAUDE.md habits

- **Don't edit CLAUDE.md mid-session** — every save invalidates. Make CLAUDE.md changes between sessions.
- **Put volatile content late** — if something changes often, prefer a skill or rule file loaded on demand over baking it into CLAUDE.md.
- **Watch total size** — bigger prefix = bigger write cost when it invalidates. The ~200-line CLAUDE.md rule (Ch 03) is also cache-friendly.

## Debugging

**"My token costs are way up."**
→ Run `/cost` to see cache read vs cache write ratio. If writes dominate, something's invalidating the prefix (CLAUDE.md edits? rule changes mid-session?).

**"Sessions got slow after the April update."**
→ TTL change. Either keep sessions tighter (less idle) or investigate cost-vs-latency on your workload [4].

## Key takeaway

**Treat the context window and prompt cache as one system.** Stable prefix + volatile suffix is the layout that caches well, and the same layout (short CLAUDE.md + on-demand rules + skills) also produces best adherence. Cost, speed, and quality all point the same direction.

## See Also

- [`03-claude-md-discipline.md`](03-claude-md-discipline.md) — CLAUDE.md size as cache input
- [`22-cost-cache.md`](22-cost-cache.md) — Production cost engineering
- [`../agents/09-context-and-cache-engineering.md`](../agents/09-context-and-cache-engineering.md) — General theory

## Sources

[4] Claude Code CHANGELOG — <https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md>
[6] Effective Context Engineering for AI Agents — <https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents>
[8] Shankar — "How I Use Every Claude Code Feature" — <https://blog.sshh.io/p/how-i-use-every-claude-code-feature>
