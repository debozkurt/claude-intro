# Commands Cheatsheet

Quick reference for Claude Code CLI flags, slash commands, and keyboard shortcuts.

---

## CLI flags (outside a session)

| Command | What it does |
|---|---|
| `claude` | Launch interactive TUI in the current directory |
| `claude --version` | Print version |
| `claude -p "prompt"` | **Headless mode** — run one prompt, print output, exit |
| `claude --continue` | Resume the most recent session in this directory |
| `claude --resume` | Pick from recent sessions (interactive picker) |
| `claude --model opus` | Start with a specific model |
| `claude --permission-mode plan` | Start in plan mode |
| `claude --permission-mode acceptEdits` | Start with auto-approved edits |
| `claude --dangerously-skip-permissions` | **YOLO** — skip every permission prompt |

---

## Slash commands (inside a session)

| Command | What it does |
|---|---|
| `/help` | List every slash command available in this session |
| `/login` | Authenticate via browser (Claude Pro/Max/Team) |
| `/logout` | Clear stored credentials |
| `/model` | Switch model (Opus ↔ Sonnet ↔ Haiku) |
| `/memory` | Open CLAUDE.md files in your editor |
| `/compact` | Summarize older turns to free context window space |
| `/clear` | Wipe conversation, keep CLAUDE.md |
| `/agents` | Manage subagents |
| `/hooks` | Manage hooks |
| `/config` | Open settings.json |
| `/cost` | Session token usage + cost |
| `/rewind` | Rewind to an earlier turn (same as Esc-Esc) |

---

## Keyboard shortcuts (inside a session)

| Key | Action |
|---|---|
| `Enter` | Send message |
| `Shift+Enter` | Newline in input |
| `Shift+Tab` | Cycle permission modes (default → acceptEdits → plan) |
| `Esc` | Interrupt the current turn |
| `Esc` twice | Open rewind picker |
| `Ctrl+C` | Exit Claude Code |
| `Ctrl+D` | Exit Claude Code (on empty input) |
| `!` prefix in input | Run the rest as a shell command in this session |

---

## Reasoning-budget keywords

Drop these into your prompt to increase the model's thinking budget:

| Keyword | Effort bump |
|---|---|
| `think` | small |
| `think hard` | medium |
| `think harder` / `megathink` | large |
| `ultrathink` | maximum |

Use for: hard debugging, architecture decisions, subtle correctness questions. Skip for: simple edits, straightforward lookups.

---

## Headless mode idioms

```bash
# One-shot question
claude -p "list every TODO comment in src/ with file:line"

# Pipe output
claude -p "summarize the last 10 commits" > release-notes.md

# Combine with other tools
claude -p "what CI does this repo use?" | grep -i github

# Use in scripts
if claude -p "are there any failing tests?" | grep -qi yes; then
  echo "Tests failing"
fi
```

---

## Worktree workflow (parallel Claude sessions)

```bash
# From your main repo:
git worktree add ../feat-auth feature/auth
cd ../feat-auth
claude                          # independent session on feature/auth

# Clean up when done:
cd ../main-repo
git worktree remove ../feat-auth
```

Each worktree:
- Has its own working tree state
- Has its own session history (stored under `~/.claude/projects/<worktree-path-slug>`)
- Can be in a different permission mode

---

## Special in-input prefix

- `!` at the start of input runs the rest as a shell command in this session. Useful for `! gcloud auth login` or anything interactive you don't want Claude to proxy.
