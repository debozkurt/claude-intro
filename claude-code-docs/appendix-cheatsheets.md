# Appendix — Cheat Sheets

*Last verified: 2026-04-19 — always verify against [4] for current state*

Quick-reference tables for CLI, slash commands, keyboard shortcuts, hook events, and `settings.json` keys.

---

## CLI flags

| Flag | What it does |
|---|---|
| `claude` | Launch interactive TUI in current dir |
| `claude --version` | Print version |
| `claude -p "prompt"` | Headless — one-shot, print output, exit |
| `claude --continue` | Resume most recent session in this dir |
| `claude --resume` | Interactive session picker |
| `claude --model opus` | Start with a specific model |
| `claude --permission-mode plan` | Start in plan mode |
| `claude --permission-mode acceptEdits` | Start with auto-accept edits |
| `claude --dangerously-skip-permissions` | Bypass all permission prompts |
| `claude --add-dir <path>` | Include another dir in working scope |

---

## Slash commands

| Command | What it does |
|---|---|
| `/help` | List every slash command |
| `/login` / `/logout` | Auth |
| `/model` | Switch model (Opus / Sonnet / Haiku) |
| `/memory` | Browse / edit loaded CLAUDE.md + rules + auto-memory |
| `/permissions` | Manage allow / deny / ask rules |
| `/compact` | Summarize older turns to free context |
| `/clear` | Wipe conversation, keep CLAUDE.md |
| `/agents` | Manage subagents |
| `/hooks` | List / manage hooks |
| `/mcp` | List MCP servers + their exposed tools |
| `/config` | Open settings.json |
| `/cost` | Session token + cost usage |
| `/rewind` | Rewind to an earlier turn (same as `Esc`-`Esc`) |
| `/branch` | Fork the current session (formerly `/fork` — renamed v2.1.77) |

Type `/` in session for the full live list.

---

## Keyboard shortcuts (in TUI)

| Key | Action |
|---|---|
| `Enter` | Send message |
| `Shift+Enter` | Newline |
| `Shift+Tab` | Cycle permission modes (Normal → Auto-accept edits → Plan) |
| `Esc` | Interrupt current turn |
| `Esc` twice | Open rewind picker |
| `Ctrl+C` / `Ctrl+D` | Exit |
| `!` prefix in input | Run the rest as a shell command in this session |

---

## Reasoning-budget keywords

Drop in prompt to increase thinking depth:

| Keyword | Effort |
|---|---|
| `think` | small |
| `think hard` | medium |
| `think harder` / `megathink` | large |
| `ultrathink` | maximum |

Not a formal API — practitioner convention the harness recognizes [8]. Ch 21.

---

## Hook events

| Event | When | Can short-circuit? |
|---|---|---|
| `PreToolUse` | Before any tool call | **Yes** — non-zero exit blocks the call |
| `PostToolUse` | After any tool call | No |
| `UserPromptSubmit` | Before Claude sees user's message | Via stdout injection |
| `SessionStart` | Session begins | No |
| `Stop` | Session ends normally | No |
| `SubagentStop` | Subagent ends | No |
| `Notification` | TUI notification fires | No |
| `PreCompact` | Before `/compact` runs | No |

Registration shape (all events):

```json
{ "hooks": { "<EventName>": [{
    "matcher": "<tool-name-or-pattern>",
    "hooks": [{ "type": "command", "command": "<shell>" }]
}]}}
```

---

## `settings.json` key reference

Top-level keys (check [1] for exhaustive current list):

```json
{
  "permissions": {
    "allow": ["Read", "Bash(npm test:*)"],
    "deny":  ["Bash(rm -rf*)"],
    "ask":   ["Bash(git push:*)"]
  },
  "autoMemoryEnabled": true,
  "autoMemoryDirectory": "~/.claude/custom-memory/",
  "mcpServers": { ... },
  "hooks": { ... },
  "env": { ... },
  "claudeMdExcludes": ["path/glob/**"]
}
```

**Settings precedence:**

```
~/.claude/settings.json              ← user
<repo>/.claude/settings.json         ← team (committed)
<repo>/.claude/settings.local.json   ← personal-project (gitignored)
```

Permission arrays (`allow`/`deny`/`ask`) are **unioned** across layers — deny from any layer wins. Other keys: later layer overrides earlier.

---

## Permission pattern syntax

| Pattern | Matches |
|---|---|
| `Read` | Whole tool |
| `Edit` | Whole tool |
| `Bash(npm test)` | Exact command `npm test` (no args) |
| `Bash(npm test:*)` | `npm test` + any args |
| `Bash(rm -rf*)` | Literal `rm -rf` prefix, any suffix |
| `Bash(git:*)` | Any `git` subcommand |
| `Read(.env)` | Exact file `.env` |
| `Read(.env*)` | `.env`, `.env.local`, etc. |

Precedence: `deny > ask > allow > mode default`.

---

## Key file locations

```
~/.claude/
├── CLAUDE.md                  ← user memory
├── settings.json              ← user settings
├── commands/<name>.md         ← user slash commands
├── skills/<name>/SKILL.md     ← user skills
├── agents/<name>.md           ← user subagents
├── rules/                     ← user rules (optional)
└── projects/<slug>/
    ├── <uuid>.jsonl           ← session files
    └── memory/MEMORY.md       ← auto-memory

<repo>/
├── CLAUDE.md                  ← project memory
└── .claude/
    ├── settings.json          ← team settings
    ├── settings.local.json    ← personal overrides (gitignored)
    ├── commands/              ← project slash commands
    ├── skills/                ← project skills
    ├── agents/                ← project subagents
    ├── rules/                 ← project rules
    └── .mcp.json              ← project MCP servers
```

---

## Sources

[1] Claude Code Docs — <https://code.claude.com/docs/en/>
[4] Claude Code CHANGELOG — <https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md>
[8] Shankar — "How I Use Every Claude Code Feature" — <https://blog.sshh.io/p/how-i-use-every-claude-code-feature>
