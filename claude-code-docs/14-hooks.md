# Chapter 14 — Hooks: Deterministic Lifecycle

*Last verified: 2026-04-19 — Prerequisites: Ch 07 — Status: Extensions*

**Fresh ground** — no prior `agents/` coverage of lifecycle hooks as a first-class primitive.

---

## Concept

Hooks are **shell commands the harness runs at lifecycle events** [1]. No model involvement — the system fires them deterministically. Register in `settings.json`; the harness guarantees execution when the event fires. If hooks return non-zero exit codes on `PreToolUse`, the tool call is short-circuited — your shell script just vetoed Claude.

This is the primitive that lets you enforce behavior *beyond* what a model might ignore. CLAUDE.md is advisory (~80% adherence); hooks are 100%. When something must happen, use a hook.

## How it works

### The taxonomy

Registered by event type [1]:

| Event | Fires | Common use |
|---|---|---|
| `PreToolUse` | Before any tool call | Enforce safety, log, short-circuit |
| `PostToolUse` | After any tool call | Notify, audit, post-process |
| `UserPromptSubmit` | Before Claude sees your message | Preprocess input, inject context |
| `SessionStart` | Session begins | Preamble, env check |
| `Stop` | Session ends normally | Teardown, notification |
| `SubagentStop` | Subagent run ends | Per-subagent telemetry |
| `Notification` | TUI notification fires | Bridge to desktop/Slack/etc |
| `PreCompact` | Before `/compact` runs | Preserve state before summarization |

### Registration shape

```json
{
  "hooks": {
    "<EventName>": [{
      "matcher": "<tool-name-or-pattern>",
      "hooks": [{ "type": "command", "command": "<shell-command>" }]
    }]
  }
}
```

The `matcher` field scopes the hook to specific tools (e.g., `"Bash"`, `"Edit"`, `"*"` for all). Lives in `~/.claude/settings.json` (user) or `<repo>/.claude/settings.json` (project). Multiple hooks can register for the same event — all fire.

### Short-circuiting

For `PreToolUse`, a non-zero exit code blocks the tool call. Use this for hard guardrails:

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "case \"$CLAUDE_TOOL_INPUT\" in *'rm -rf'*) exit 1 ;; esac"
      }]
    }]
  }
}
```

Now Claude cannot run `rm -rf` regardless of permission mode.

### Environment available

Hooks receive tool call details via environment variables (`CLAUDE_TOOL_INPUT`, `CLAUDE_FILE_PATHS`, session metadata). Check the docs [1] for the full env list per event.

## Why it matters

Three unique capabilities that no other primitive offers:

1. **Deterministic enforcement** — "Every time the Edit tool runs, also run ESLint" isn't a preference for Claude, it's a system rule. Hooks deliver this.
2. **Observability** — hooks emit telemetry the model never sees. Log to a file, ship to Datadog, send a Slack, emit OpenTelemetry spans.
3. **Trust boundary** — a `PreToolUse` hook that rejects dangerous commands is stronger than `deny` in permissions, because it can run arbitrary logic (not just pattern matching).

## Patterns

**Safety guardrail** — block destructive commands with logic more nuanced than `deny`:

```json
{ "hooks": { "PreToolUse": [{ "matcher": "Bash",
  "hooks": [{ "type": "command",
    "command": "~/bin/safety-check.sh \"$CLAUDE_TOOL_INPUT\" || exit 1"
  }]}]}}
```

**Telemetry emitter** — log every tool call for audit:

```json
{ "hooks": { "PreToolUse": [{ "matcher": "*",
  "hooks": [{ "type": "command",
    "command": "echo \"$(date -Iseconds) $CLAUDE_TOOL_NAME $CLAUDE_TOOL_INPUT\" >> ~/.claude-audit.log"
  }]}]}}
```

**Notification on completion** — fire a desktop/Slack notification when a long skill finishes:

```json
{ "hooks": { "PostToolUse": [{ "matcher": "Write",
  "hooks": [{ "type": "command",
    "command": "grep -l 'Triggered by:' $CLAUDE_FILE_PATHS 2>/dev/null && osascript -e 'display notification \"Lesson saved\"'"
  }]}]}}
```

(This is the pattern in the composition example from the primer — PostToolUse on `Write`, filtered by a content marker.)

**Pre-prompt augmentation** — inject env context on every user prompt:

```json
{ "hooks": { "UserPromptSubmit": [{ "matcher": "*",
  "hooks": [{ "type": "command",
    "command": "echo \"Current branch: $(git branch --show-current)\""
  }]}]}}
```

The stdout of the hook becomes additional context injected into Claude's view.

## Why not `.claude/hooks/`?

Hooks live in `settings.json`, not a separate dir like skills or agents. Why? Because they're *system config*, not *content*. The shell commands they fire *can* live as scripts elsewhere (`~/bin/safety-check.sh`), but the registration is a settings concern.

## Debugging

**"Hook isn't firing."**
→ Verify: (a) event name is exact (`PreToolUse`, not `PreToolUsed`), (b) matcher actually matches, (c) settings file is valid JSON (missing comma breaks silent), (d) the command actually runs when invoked manually. Hooks fail silently by design.

**"Hook fires but doesn't short-circuit."**
→ Only `PreToolUse` can short-circuit via non-zero exit. Other events just run.

**"Too many hooks, too slow."**
→ Every hook adds latency to the tool call. Audit your hook list with `/hooks`. Consolidate or gate.

## Key takeaway

**Hooks are where "every time X, do Y" actually belongs.** Memory/CLAUDE.md is advisory — Claude may ignore. Hooks are system-executed — they always run. For safety guardrails (block `rm -rf`), observability (audit log every Bash), or integration (notify on completion), hooks are the correct primitive.

## See Also

- [`07-permission-modes.md`](07-permission-modes.md) — Permissions as the first-pass gate; hooks as the second
- [`23-observability.md`](23-observability.md) — Hooks as telemetry emitters in production
- [`24-security.md`](24-security.md) — Hooks as security defense in depth
- [`25-decision-tree.md`](25-decision-tree.md) — Hook vs skill vs rule

## Sources

[1] Claude Code Docs — hooks — <https://code.claude.com/docs/en/hooks>
[2] Claude Code Best Practices — <https://www.anthropic.com/engineering/claude-code-best-practices>
