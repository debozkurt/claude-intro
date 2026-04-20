# Chapter 21 — `ultrathink` and Thinking Budgets

*Last verified: 2026-04-19 — Prerequisites: Ch 01 — Status: Advanced*

**Builds on:** [`../agents/02-anatomy-of-an-llm-call.md`](../agents/02-anatomy-of-an-llm-call.md) (reasoning tokens and thinking budget).

---

## Concept

Claude's reasoning-enabled models allocate a **thinking budget** — internal tokens the model spends reasoning before producing its answer. Larger budget = deeper reasoning = better results on hard problems, at the cost of latency and tokens [12]. Claude Code gives you two ways to influence this: the **official `thinking_budget` parameter** in the SDK, and **practitioner magic strings** like `ultrathink` in the CLI.

The `ultrathink` string isn't a formally documented feature — it's a practitioner convention [8] that signals maximum reasoning budget. The official knob is in the SDK or via model settings. Both matter; know the difference.

## How it works

### The model-side mechanism

Modern Claude models (Opus, Sonnet 4.X) support extended thinking: the model allocates up to N internal tokens for chain-of-thought reasoning before generating the visible answer [12]. The ratio of thinking-to-answer varies by model and request; on hard problems, thinking can be an order of magnitude larger than the answer.

Three budget regimes:

| Budget | Behavior | Use for |
|---|---|---|
| Off / minimal | Standard generation — no extended thinking | Simple edits, fast loops |
| Medium | Moderate reasoning depth | Typical coding tasks |
| Maximum | Deep reasoning; may take 30s+ before answering | Architecture, debugging, subtle correctness |

### The practitioner strings

Drop these in your prompt [8]:

| Keyword | Effort bump (empirical) |
|---|---|
| `think` | small |
| `think hard` | medium |
| `think harder` / `megathink` | larger |
| **`ultrathink`** | maximum |

These are *hints* Claude Code's internals translate to thinking-budget settings. They work because the harness recognizes them and sets budget accordingly — not because the model itself sees the magic word. Caveat: this is practitioner knowledge, not a formal API contract.

### The SDK side

For programmatic use, set the budget explicitly:

```python
result = run_agent(
    prompt="...",
    thinking_budget=8192,  # tokens for reasoning
)
```

The SDK and CLI both ultimately set the same model-side parameter; the CLI just offers the string-hint convenience.

## Why it matters

Extended thinking pays dramatic dividends on three task classes:

1. **Debugging** — especially intermittent, multi-variable, or subtly-wrong bugs where the first hypothesis is usually wrong
2. **Architecture decisions** — weighing tradeoffs across approaches, considering failure modes, surfacing hidden constraints
3. **Code correctness** — race conditions, edge cases in concurrent code, subtle API misuses

For straightforward work (rename a variable, add a route following existing patterns), extended thinking is a tax. The pattern is opt-in, not default-on.

## When to use

**Reach for it when:**

- The problem has 2+ plausible root causes (debugging)
- You're choosing between architectural approaches (design)
- The code touches concurrency, security, or correctness-sensitive areas
- You've tried once without, gotten a bad answer, and now need depth

**Don't reach for it when:**

- Simple, pattern-based tasks (add route, rename symbol)
- You're iterating fast (latency matters more than depth)
- Conversational exploration (you want quick back-and-forth, not deep answers)

## Patterns

### Explicit in prompt

```
ultrathink — our webhook deliveries drop 2% at peak. Here's the queue:
@src/queue.ts @src/worker.ts
What's the most likely root cause? Consider failure modes I might miss.
```

### Via plan mode compatibility

Plan mode + `ultrathink` together = thorough pre-execution reasoning. The plan you get back reflects real consideration, not just sketch-level.

### Worktree + ultrathink for hard refactors

```bash
git worktree add ../refactor-attempt feat/bigrefactor
cd ../refactor-attempt
claude
# in session:
# ultrathink — refactor the storage layer. Consider: backpressure,
# transaction boundaries, rollout safety. Produce a plan.
```

Combined with plan mode + rewind, this is the "think carefully before touching production code" workflow.

## The contested framing

Practitioners [8] report ultrathink as transformative for hard problems — claims of "catches bugs I would have missed" and "finds real architectural issues." Anthropic's official framing is more measured: extended thinking helps when the task genuinely requires it; over-use costs without benefit.

Practical advice: default off. Learn when to reach for it by noticing which tasks frustrate you without. Over weeks, the pattern crystallizes.

## Debugging

**"`ultrathink` doesn't seem to help."**
→ Task may not benefit — it's not a universal depth multiplier. Also check: is it actually triggering? Some versions of Claude Code log the thinking budget per turn; verify it's set.

**"Latency jumped."**
→ Expected — deep thinking takes time. That's why it's opt-in.

**"Cost jumped more than I expected."**
→ Thinking tokens bill at regular input-token rates. `ultrathink` on a complex prompt can 5×+ token cost for that turn. See `/cost`.

## Key takeaway

**`ultrathink` is opt-in deep reasoning for the cases that need it.** Magic string in the CLI, `thinking_budget` in the SDK — same underlying knob. Reach for it on debugging, architecture, and correctness-sensitive work. Leave it off for straightforward edits. The pattern that separates master-class users: *recognizing which task class benefits* without reflexively over-thinking.

## See Also

- [`08-plan-mode.md`](08-plan-mode.md) — Combines well with ultrathink
- [`22-cost-cache.md`](22-cost-cache.md) — Cost implications
- [`../agents/02-anatomy-of-an-llm-call.md`](../agents/02-anatomy-of-an-llm-call.md) — Reasoning tokens in agent theory

## Sources

[8] Shankar — "How I Use Every Claude Code Feature" — <https://blog.sshh.io/p/how-i-use-every-claude-code-feature>
[12] Advanced tool use (extended thinking context) — <https://www.anthropic.com/engineering/advanced-tool-use>
