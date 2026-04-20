# Permissions Cheatsheet

How to configure `settings.json` so Claude Code interrupts you as little as possible, while still blocking the real footguns.

---

## Where settings live

| Path | Scope | Committed? |
|---|---|---|
| `~/.claude/settings.json` | User — applies to every project | No |
| `<repo>/.claude/settings.json` | Project — team-shared | Yes |
| `<repo>/.claude/settings.local.json` | Your personal overrides for this project | **No** (gitignore it) |

**Merge rule:** later layer wins for the same key. Permission arrays (`allow`/`deny`/`ask`) are unioned across layers.

---

## The three permission modes

| Mode | Behavior | When to use |
|---|---|---|
| `default` | Ask before each tool call | Learning, unfamiliar task, risky repo |
| `acceptEdits` | Auto-approve `Read`/`Edit`/`Write`; ask for the rest | Editing flow in a trusted project |
| `bypassPermissions` | Approve everything (**YOLO**) | Sandboxed/throwaway env only |

Cycle live with `Shift+Tab`. Start in a specific mode with `--permission-mode <name>`.

---

## Precedence inside `permissions`

```
deny  >  ask  >  allow
```

If a tool call matches any `deny` pattern, it's blocked — no prompt.
Else if it matches any `ask` pattern, you're prompted.
Else if it matches any `allow` pattern, it runs silently.
Otherwise falls through to the active permission mode's default.

---

## Pattern syntax

- Bare tool name — the whole tool: `"Read"`, `"Edit"`, `"Grep"`
- `Bash(<glob>)` — shell command matching: `"Bash(npm test:*)"`, `"Bash(rm -rf*)"`
- `*` glob matches anything including spaces/slashes
- `:*` after a command matches any args to that command

---

## Starter `settings.json` — sensible defaults

Drop this in `~/.claude/settings.json` for a reasonable baseline:

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Grep",
      "Glob",
      "Edit",
      "Write",
      "Bash(ls:*)",
      "Bash(cat:*)",
      "Bash(pwd)",
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git branch:*)",
      "Bash(npm test:*)",
      "Bash(npm run:*)",
      "Bash(pytest:*)",
      "Bash(node:*)",
      "Bash(python:*)"
    ],
    "deny": [
      "Bash(rm -rf*)",
      "Bash(sudo:*)",
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(git push --force*)",
      "Bash(git reset --hard*)"
    ],
    "ask": [
      "Bash(git commit:*)",
      "Bash(git push:*)",
      "Bash(npm install:*)",
      "Bash(npm publish:*)"
    ]
  }
}
```

**Rationale:**
- Reading/searching is always free — Claude should never ask to `ls` or `grep`.
- Running tests is free — you *want* Claude to validate its own work.
- Shell primitives that destroy state are denied outright.
- Git state changes (commit/push/install) are asked — you want to know.

---

## Per-project overrides

In `<repo>/.claude/settings.json` (committed), narrow or broaden for your team:

```json
{
  "permissions": {
    "allow": [
      "Bash(cargo build:*)",
      "Bash(cargo test:*)",
      "Bash(cargo check:*)"
    ],
    "deny": [
      "Bash(cargo publish:*)"
    ]
  }
}
```

In `<repo>/.claude/settings.local.json` (gitignored), your personal loosening:

```json
{
  "permissions": {
    "allow": ["Bash(git push:*)"]
  }
}
```

---

## YOLO safety pattern

Never run `--dangerously-skip-permissions` on your host shell in a real project. Use Docker:

```bash
docker run -it --rm \
  -v "$PWD:/work" \
  -w /work \
  -e ANTHROPIC_API_KEY \
  node:20 bash

# inside the container:
npm install -g @anthropic-ai/claude-code
claude --dangerously-skip-permissions
```

The container has no access to your SSH keys, cloud CLI creds, or shell history. When it dies, damage is bounded to the mounted volume.

**Still don't mount your `~/.ssh`, `~/.aws`, or `~/.config` directories into a YOLO container.** Just the project folder.

---

## Debugging permission prompts

**"Why did Claude ask me X?"**
- Check `<repo>/.claude/settings.json` + `~/.claude/settings.json` for an `ask` pattern matching X
- Or: X isn't in any list, and the active mode is `default`

**"Why won't Claude run Y? It just gets blocked."**
- A `deny` pattern somewhere matches Y
- Check all three layers: global user, project, project-local

**"I keep getting prompted for `Bash(git status)`."**
- Your `allow` pattern is probably `"Bash(git status)"` without the `:*`
- Fix: `"Bash(git status:*)"` (the `:*` matches any args)
