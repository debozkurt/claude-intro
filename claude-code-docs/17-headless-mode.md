# Chapter 17 — Headless Mode

*Last verified: 2026-04-19 — Prerequisites: Ch 01 — Status: Headless & SDK*

**Fresh ground** — the CLI's non-interactive face.

---

## Concept

Headless mode runs Claude Code as a **one-shot, non-interactive CLI** via `claude -p "task"` [1]. Input is a single prompt; output is streamed to stdout; exit when done. No TUI, no session picker, no interactive approval — the session runs to completion and the process exits.

This is what turns Claude Code from a chat app into a **composable Unix tool**. Everything you can do in a terminal with `grep`, `jq`, and pipes now works with Claude as another station on the line.

## How it works

### Basic invocation

```bash
claude -p "list every TODO comment in src/ with file and line"
claude -p "summarize the last 10 commits" > release-notes.md
claude -p "does this repo have a CI config? which CI?" | grep -i github
```

Claude reads the prompt, uses tools as needed, prints the final answer to stdout, exits. Exit code indicates success/failure like any other CLI.

### Permission behavior

By default, headless mode respects your `settings.json` permissions. For automation (CI, cron, scripts), you have options:

- **Pre-approve via `settings.json`** — `allow` list covers everything the script will do. Safe.
- **`--dangerously-skip-permissions`** — YOLO for sandboxed/CI contexts. Do this only in environments where nothing of value lives.
- **`--permission-mode plan`** — returns a plan as text; you can approve manually elsewhere.

### Streaming output

Headless can stream structured JSON for pipelines that parse Claude's output turn-by-turn. Check [1] for current flags (the specific shape has evolved; common is `--output-format json` or streaming SSE). Useful when a downstream tool (e.g., Datadog, a dashboard) wants incremental updates.

### Context loading

Headless still reads CLAUDE.md, rules, auto-memory — everything loads like an interactive session [1]. The permission context comes from the user running the command. In CI, that's a service user with its own `~/.claude/` — plan accordingly.

## Why it matters

Three categories unlock:

1. **Pipelines and scripts** — Claude becomes just another command. Pipe, redirect, compose. `grep + jq + claude` is a valid stack.
2. **CI integration** — run a skill on every PR, run a security-review subagent on merge, generate release notes from commits. Headless is the mechanism.
3. **Scheduled and remote execution** — cron + headless = scheduled Claude. Webhook + headless (deployed to a server) = "Claude on demand." This is the foundation for remote control (Ch 20 covers patterns).

## Patterns

### Conditional in scripts

```bash
if claude -p "are there any failing tests in this repo?" | grep -qi "yes"; then
  echo "Tests failing — skipping deploy"
  exit 1
fi
```

### CI audit on PRs

```yaml
# .github/workflows/claude-review.yml
- name: Claude security review
  run: |
    git diff origin/main..HEAD | \
      claude -p "review this diff for security issues; exit 1 if any critical"
```

### Nightly cron

```cron
0 2 * * * cd ~/projects/my-app && \
  claude -p "scan for TODO/FIXME added this week; summarize" \
  | mail -s "Nightly TODO report" me@example.com
```

### Pipe into formatted output

```bash
claude -p "list all API endpoints in this repo; output as JSON array of {path, method}" \
  | jq '.[] | select(.method == "POST")'
```

## Design heuristics for headless prompts

Headless has no back-and-forth, so the prompt must be **self-contained**. Rules:

- **Specify the output format** — "output as JSON", "one per line", "markdown table". Without guidance, Claude defaults to conversational prose, which is bad for pipes.
- **Constrain scope** — "check only `src/`" not "check the repo."
- **Name the exit condition** — "exit 0 if no issues, 1 otherwise."
- **Pre-approve tools** — ensure `settings.json` has what the task needs in `allow`.

## Debugging

**"Headless just hung."**
→ Permission prompt likely fired — but there's no TUI to respond. Either pre-approve in `settings.json`, use `--dangerously-skip-permissions` (sandboxed), or switch to `plan` mode.

**"Output wasn't what I expected."**
→ Prompt wasn't specific about format. Add explicit "output shape" instructions.

**"Cost is higher than interactive."**
→ Headless re-loads CLAUDE.md and rules every invocation — no cache warmth from prior runs. Batch work where possible; cache discipline (Ch 06) matters even more.

## Key takeaway

**Headless turns Claude Code into a Unix citizen.** The interactive TUI is for exploration; headless is for automation. Scripts, CI, cron, webhooks — anywhere you'd normally run a command, you can now run Claude. Specify output shape, pre-approve tools, and it composes like any other CLI.

## See Also

- [`18-agent-sdk.md`](18-agent-sdk.md) — The SDK as the library flavor of this same primitive
- [`20-long-running-claude.md`](20-long-running-claude.md) — Headless as the foundation for long-running loops
- [`23-observability.md`](23-observability.md) — Telemetry for headless runs

## Sources

[1] Claude Code Docs — <https://code.claude.com/docs/en/>
[8] Shankar — "How I Use Every Claude Code Feature" — <https://blog.sshh.io/p/how-i-use-every-claude-code-feature>
