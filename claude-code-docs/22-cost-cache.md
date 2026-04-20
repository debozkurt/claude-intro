# Chapter 22 — Cost and Cache Engineering

*Last verified: 2026-04-19 — Prerequisites: Ch 06, Ch 21 — Status: Production*

**Builds on:** [`06-context-cache.md`](06-context-cache.md) · [`../agents/21-cost-and-latency.md`](../agents/21-cost-and-latency.md) (cost/latency in agents).

---

## Concept

Cost is the production concern that decides whether Claude Code *stays* a daily-use tool or becomes an occasional treat. Three levers — **model selection**, **cache discipline**, and **context layout** — together determine whether a heavy user pays $20/month or $2000/month for the same work [6][8].

Master-class framing: cost is a function of three ratios — cache read vs write, model tier, and context turnover rate. Tuning any one moves the bill materially. Tuning all three moves it dramatically.

## How it works

### The lever: model selection

Claude Code supports Opus, Sonnet, Haiku, switchable mid-session via `/model`. Relative economics (2026 pricing, always verify latest):

| Model | Relative cost | Best for |
|---|---|---|
| Opus | ~1.0x | Architecture, debugging, subtle correctness |
| Sonnet | ~0.2x | Default for most work |
| Haiku | ~0.05x | Simple edits, pattern-based work, bulk processing |

The reflex is Opus-everywhere; the practitioner move is Haiku-for-Ralph-loops, Sonnet-for-normal, Opus-for-hard-problems. Switching mid-session via `/model` is zero-friction once you develop the taste.

### The lever: cache discipline

From Ch 06: cache hits are ~10% the cost of cache writes [6]. The stable prefix (system prompt + CLAUDE.md + rules + skill frontmatter) is what caches; the volatile suffix (current turn + tool results) always writes.

**Cache-friendly habits:**

- Short CLAUDE.md (< 200 lines) — smaller prefix, cheaper writes when invalidated
- Don't edit CLAUDE.md mid-session — edit between sessions, preserves warmth
- Long-ish sessions over many short ones — amortize the prefix write
- Respect the 5-min TTL — keep turns < 5 min apart for warmth [4]

**Cache-unfriendly habits:**

- Path-scoped rules that load mid-session — technically invalidate partial cache
- Skill invocation adds skill body to prefix — invalidates until the skill completes
- CLAUDE.md tweaks during a session — every edit is a new write

### The lever: context turnover

Every `/compact` or `/clear` restarts some cache warmth. Every new session restarts all of it. Minimize if you're watching cost:

- Prefer long sessions with `/compact` over many short ones
- But don't let sessions balloon to 200+ turns — diminishing returns, degraded adherence
- Sweet spot: ~30-60 turn sessions, `/compact` at natural breakpoints, `/clear` when truly switching tasks

### The April 2026 TTL change

Still actively contested in the community [4]. The change: cache TTL default dropped from 1 hour to 5 minutes for many Claude Code requests. Practical effect on interactive users:

- Slow, human-paced sessions (5+ min between prompts) now pay more
- Fast turn-by-turn work is unaffected
- Background / scheduled / Ralph loops can be dramatically cheaper *or* more expensive depending on timing

Check current state in the CHANGELOG [4] — this is the kind of thing that shifts. Some users explicitly set longer TTLs via flags where supported.

## Practical patterns

### Cost-aware Ralph loop

```python
# Use Haiku for cheap pattern-matching passes
fast_pass = run_agent(prompt="...", model="haiku-4")
# Promote to Sonnet only where fast_pass flagged concern
if fast_pass.has_concerns:
    detailed = run_agent(prompt=f"Review: {fast_pass.concerns}", model="sonnet-4-6")
```

Two-tier passes: cheap model screens, expensive model drills in only where needed. 10-20× cost savings on bulk processing.

### Mid-session model switch

During a session:

- Start in Sonnet for exploration
- `/model opus` when you hit a hard debugging moment (Ch 21)
- `/model sonnet` back after you've solved it
- Never bulk-process in Opus

### Cost observability

`/cost` shows session token use and cost estimate. Habits:

- Check `/cost` at session end as a feedback loop — which sessions were expensive? why?
- For production loops, emit cost via hooks (Ch 23) for dashboards
- Set a daily / weekly budget alarm on the API key if self-paying

## Debugging

**"My bill is up 10× and nothing changed."**
→ Usually: a CLAUDE.md edit mid-session (cache invalidation) or a long auto-memory file (bigger prefix) or a change in model default. Check `/cost` per session; look for outliers.

**"Ralph loop is blowing budget."**
→ Each iteration writes cache fresh. Either: (a) use Haiku for iterations that don't need Opus, (b) batch more work per iteration, (c) accept the write cost and maximize read hits within iteration.

**"Cache hit rate is lower than it used to be."**
→ April 2026 TTL change [4]. Check if your idle times between turns are now exceeding 5 min. Faster back-and-forth or larger per-turn work reclaims warmth.

## Key takeaway

**Cost is tunable; most heavy Claude Code bills come from a few fixable habits.** Model selection matches task tier; cache discipline avoids needless writes; context turnover is kept reasonable. Run `/cost` often enough to notice outliers; adjust one lever at a time. Cost-aware usage is the difference between Claude Code being a tool you use constantly and one you ration.

## See Also

- [`06-context-cache.md`](06-context-cache.md) — Cache mechanics in detail
- [`23-observability.md`](23-observability.md) — Cost as first-class telemetry
- [`../agents/21-cost-and-latency.md`](../agents/21-cost-and-latency.md) — General cost theory

## Sources

[4] Claude Code CHANGELOG — <https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md>
[6] Effective Context Engineering — <https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents>
[8] Shankar — "How I Use Every Claude Code Feature" — <https://blog.sshh.io/p/how-i-use-every-claude-code-feature>
