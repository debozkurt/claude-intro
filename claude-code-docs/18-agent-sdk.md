# Chapter 18 — The Agent SDK

*Last verified: 2026-04-19 — Prerequisites: Ch 17 — Status: Headless & SDK*

**Builds on:** [`17-headless-mode.md`](17-headless-mode.md). **Fresh ground** — the library-as-harness framing.

---

## Concept

The **Agent SDK** (formerly "Claude Code SDK," renamed late 2025 to reflect general-purpose agent utility) exposes Claude Code's harness — loop, tools, context management, subagents, MCP — as a Python or TypeScript library [5]. The CLI (`claude`, `claude -p`) and the SDK are **two faces of the same primitive**. Anything the CLI does, the SDK can do programmatically, and vice versa. Same harness, different I/O.

Master-class framing: if you understand Claude Code as an agent harness (Ch 01), the SDK is the moment it becomes obvious the harness isn't *tied to* the CLI — it's a general-purpose Claude-agent runtime that *ships with* a CLI.

## How it works

### Install

**Python:**

```bash
pip install claude-agent-sdk
```

**TypeScript:**

```bash
npm install @anthropic-ai/claude-agent
```

### Minimal usage — Python

```python
from claude_agent_sdk import run_agent

result = run_agent(
    prompt="list TODO comments in src/",
    cwd="./my-project",
    permission_mode="acceptEdits",
    allowed_tools=["Read", "Grep"],
)
print(result.final_message)
```

The SDK loads the same CLAUDE.md, rules, permissions, MCP config — because it reads the same on-disk state (Ch 02). It's not a separate Claude; it's *Claude Code without the TUI*.

### Subagents in code

The SDK exposes subagent delegation programmatically [5]:

```python
from claude_agent_sdk import Agent

researcher = Agent.from_file("~/.claude/agents/tutor-researcher.md")
findings = researcher.invoke("Research SIP early media semantics")
# findings is structured markdown — same shape as the CLI would receive
```

### Streaming and callbacks

SDK consumers need incremental updates — long agent runs shouldn't block. Both Python and TS SDKs provide streaming iterators over turns, plus callbacks for tool calls:

```python
for turn in run_agent_stream(prompt="..."):
    print(turn.type, turn.content)
    # type in: "model_thinking", "tool_call", "tool_result", "final"
```

This matches the CLI's streaming JSON output format — same data, different frontend.

## Why it matters

Three scenarios where the SDK wins over the CLI:

1. **Embedding Claude in a product** — your API endpoint gets a request, spawns a Claude agent, streams back the result. The SDK is the library; the CLI is a dev tool.
2. **Custom orchestration** — fan out N Claudes with different prompts, aggregate results, implement your own routing. The SDK gives you the harness; you build the workflow.
3. **Testing and evaluation** — run Claude against a benchmark, collect metrics, iterate on prompts / skills / rules. Programmatic access is how you evaluate seriously.

What the CLI still wins on:

- Interactive exploration — no frontend work, just `claude`
- Ad-hoc tasks — headless (`claude -p`) is simpler than a Python script
- Team-shared workflows — skills in `.claude/skills/` are easier than a library import

## The "one primitive, two frontends" insight

Consider what the CLI actually is:

```
terminal TUI  ←→  Claude Code harness  ←→  Anthropic API
```

And the SDK:

```
your Python code  ←→  Claude Code harness  ←→  Anthropic API
```

Same middle. The middle is what matters. It handles:

- Context assembly (CLAUDE.md, rules, memory)
- Tool dispatch (Read, Edit, Bash, Agent, MCP tools)
- Session state (JSONL on disk at `~/.claude/projects/<slug>/`)
- Permissions
- Subagents, hooks

The CLI and SDK are just two ways to *talk to* the harness. That's the insight that lets you bounce freely between ad-hoc CLI usage and building production systems on the same runtime.

## Patterns

### Scheduled headless via cron

```bash
0 2 * * * cd /repo && claude -p "$(cat /tmp/daily-audit-prompt.md)" | mail ...
```

### Background worker via SDK

```python
while True:
    task = queue.get()
    result = run_agent(prompt=task.prompt, cwd=task.repo)
    queue.publish_result(task.id, result)
```

### Multi-agent orchestration

```python
reviewers = [Agent.from_file(f"agents/{name}.md") for name in ["security", "perf", "style"]]
findings = [r.invoke(diff) for r in reviewers]  # parallel in practice
summary = aggregate(findings)
```

## Debugging

**"SDK behavior differs from CLI."**
→ Usually env: check CWD, env vars (tokens, MCP config), user-level settings. The SDK reads the same files; if behavior diverges, one of those is different.

**"Subagent file not found."**
→ SDK resolves paths relative to CWD unless absolute. Use `Path.expanduser()` or absolute paths for user-scoped files.

**"Slower than CLI."**
→ Cache misses. The CLI warms the cache across turns in one session; SDK one-shot calls re-pay the write each time unless you reuse sessions.

## Key takeaway

**The Agent SDK and the Claude Code CLI are the same harness.** Reach for the SDK when you're building a product, orchestrating programmatically, or testing at scale. Reach for the CLI when you're exploring, running ad-hoc tasks, or sharing workflows with teammates. Same primitives underneath — pick the frontend that fits the moment.

## See Also

- [`17-headless-mode.md`](17-headless-mode.md) — The CLI sibling of the SDK
- [`20-long-running-claude.md`](20-long-running-claude.md) — Long-running patterns on top of SDK
- [`27-recipes.md`](27-recipes.md) — Concrete SDK workflows
- [`../agents/26-reference-architecture.md`](../agents/26-reference-architecture.md) — General agent reference architecture

## Sources

[5] Agent SDK overview — <https://code.claude.com/docs/en/agent-sdk/overview> + <https://platform.claude.com/docs/en/agent-sdk/subagents>
