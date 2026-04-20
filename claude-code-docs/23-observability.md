# Chapter 23 — Observability

*Last verified: 2026-04-19 — Prerequisites: Ch 14, Ch 17 — Status: Production*

**Builds on:** [`14-hooks.md`](14-hooks.md) · [`../agents/22-observability.md`](../agents/22-observability.md) (agent observability).

---

## Concept

Claude Code running casually is a black box by default. Claude Code in production or long-running mode **must be observable** — you need to know what it did, what it cost, what tools it called, how often it succeeded. The tools for this are hooks (Ch 14), session JSONL (Ch 02), `/cost`, and OpenTelemetry for SDK deployments.

Master-class framing: observability isn't optional for serious use. If you can't answer "what did Claude do in the last hour?" in 10 seconds, you're operating blind.

## How it works

### Three telemetry sources

| Source | What you get | How to access |
|---|---|---|
| **Hooks** (Ch 14) | Every tool call, every event | Write to file / DB / endpoint in the hook's shell command |
| **Session JSONL** (Ch 02) | Full turn-by-turn history | `~/.claude/projects/<slug>/*.jsonl` — grep, jq |
| **`/cost`** | Tokens, cost per session | Inside session; TUI output |
| **OpenTelemetry** | Traces across SDK calls | `TRACEPARENT` env var honored in headless [4] |

### Hook-based telemetry

The simplest observability pattern: every tool call logged to a file.

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "echo \"$(date -u +%FT%TZ) $CLAUDE_SESSION_ID $CLAUDE_TOOL_NAME\" >> ~/.claude-audit.log"
      }]
    }]
  }
}
```

Now `tail -f ~/.claude-audit.log` shows everything Claude is doing across all sessions. Upgrade path: emit structured JSON instead of plaintext, ship to Loki/Datadog/your warehouse.

### Session JSONL replay

For debugging past sessions, the JSONL is your ground truth (Ch 02):

```bash
jq 'select(.role == "tool_use") | .tool_name' \
  ~/.claude/projects/<slug>/<session>.jsonl \
  | sort | uniq -c
```

This gives tool-call counts for that session. No extra instrumentation needed — the JSONL was always there.

### OpenTelemetry (SDK / headless)

Headless Claude Code honors `TRACEPARENT` [4]. If your caller is instrumented with OTel, the Claude agent run shows up as a span in your traces:

```python
with tracer.start_as_current_span("claude-agent"):
    result = run_agent(prompt="...")
```

For SDK deployments, this turns Claude into just another instrumented service. You get latency breakdowns per turn, tool-call duration histograms, cost attribution per caller.

### Cost observability

`/cost` in session is the cheap feedback loop. For production:

- Emit cost via `Stop` hook at session end: `echo "$CLAUDE_SESSION_ID $COST" >> cost.log`
- Aggregate in a dashboard (Grafana reading from the log)
- Set budget alarms on the API key level

## Why it matters

Four concrete risks observability mitigates:

1. **Runaway costs** — a skill or loop going haywire can burn $100s in hours. Cost telemetry catches it.
2. **Silent failures** — a headless job that "ran" but produced wrong output. JSONL replay + structured logs surface the issue.
3. **Drift** — Claude's behavior changes subtly as CLAUDE.md grows or models update. Baselines from telemetry catch drift.
4. **Security** — auditing tool calls is how you detect prompt injection exploits (Ch 24). No audit log, no detection.

## Patterns

### Per-session cost + tool-call summary

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "jq -s '{session: .[0].session_id, turns: length, tools: [.[] | select(.role == \"tool_use\") | .tool_name] | group_by(.) | map({name: .[0], count: length})}' \"$CLAUDE_SESSION_FILE\" >> session-summaries.jsonl"
      }]
    }]
  }
}
```

Every session end emits a structured summary to a dashboard-ready log.

### Skill-usage frequency

```bash
grep -h "skill_invoked" ~/.claude-audit.log | \
  awk '{print $4}' | sort | uniq -c | sort -rn
```

Which skills you actually use vs. which ones you wrote and forgot. Good input for pruning.

### Error detection

Hook that fires on `PostToolUse` when the tool result contains error patterns — ship to Slack:

```bash
if echo "$CLAUDE_TOOL_OUTPUT" | grep -qi "error\|fatal\|traceback"; then
  slack-alert "Tool error in $CLAUDE_SESSION_ID"
fi
```

### Long-running progress

For Ralph loops (Ch 20), emit heartbeat from inside the loop itself — structured log lines to a progress file. `tail -f progress.log` gives you real-time visibility. Don't rely on hooks alone for long-running; you need explicit progress beats.

## Debugging

**"Hooks fire but nothing logs."**
→ Quoting issue in the hook command. Shell escaping bites. Test the command manually with sample env vars.

**"Session JSONL is massive."**
→ Rotate / archive old sessions. Claude Code doesn't auto-prune `~/.claude/projects/`. Periodic cleanup is reasonable.

**"OTel span missing."**
→ `TRACEPARENT` env needs to be set *in the Claude Code invocation's environment*, not just in the caller's span context. Make sure your wrapper exports it.

## Key takeaway

**If you can't answer "what did Claude do and what did it cost?" you're operating blind.** Hooks for per-call audit, JSONL for session replay, `/cost` for quick feedback, OTel for production traces. The bar rises with stakes — casual use needs little; production long-running needs all four.

## See Also

- [`14-hooks.md`](14-hooks.md) — The primitive under hook-based telemetry
- [`20-long-running-claude.md`](20-long-running-claude.md) — Telemetry for long-running
- [`22-cost-cache.md`](22-cost-cache.md) — Cost as a first-class metric
- [`../agents/22-observability.md`](../agents/22-observability.md) — Agent observability theory

## Sources

[1] Claude Code Docs — <https://code.claude.com/docs/en/>
[4] Claude Code CHANGELOG (TRACEPARENT) — <https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md>
