# Chapter 02 — The On-Disk Model

*Last verified: 2026-04-19 — Prerequisites: Ch 01 — Status: Foundations*

**Builds on:** [`01-mental-model.md`](01-mental-model.md). **Fresh ground** — no prior `agents/` coverage of Claude-Code-specific layout.

---

## Concept

Claude Code stores every meaningful piece of state as **plaintext files on disk**. Understanding the layout unlocks `/rewind`, `/resume`, `/branch`, auto-memory, debugging, and the Agent SDK all at once. Nothing is hidden in a database, binary blob, or cloud — it's all `.md` and `.json` under `~/.claude/` and your repo's `.claude/`.

Master-class mindset: treat `~/.claude/projects/<slug>/` the way you treat `.git/` — mostly don't touch, but *knowing what's in there* is the difference between using Claude Code and owning it.

## How it works

### The two trees

```
~/.claude/                        # per-user, all projects
├── CLAUDE.md                    # user-level preferences
├── settings.json                # user-level permissions, hooks, env
├── commands/                    # user slash commands
├── skills/                      # user skills (SKILL.md per dir)
├── agents/                      # user subagents (one .md each)
├── rules/                       # user-level rules (optional)
├── projects/                    # per-project state (below)
└── keybindings.json             # optional

<repo>/                          # per-project, committed with code
├── CLAUDE.md                   # project-level, team-shared
├── .claude/
│   ├── settings.json           # team permissions
│   ├── settings.local.json     # personal overrides (gitignored)
│   ├── commands/               # project slash commands
│   ├── skills/                 # project skills
│   ├── agents/                 # project subagents
│   ├── rules/                  # project rules
│   └── .mcp.json               # project MCP servers (Ch 16)
```

### The projects/ directory

`~/.claude/projects/<slug>/` is where ephemeral-but-recoverable state lives. The `<slug>` is your project's absolute path with `/` → `-`. `~/Downloads` becomes `-Users-you-Downloads`.

Inside each project slug dir:

```
~/.claude/projects/<slug>/
├── <session-uuid>.jsonl        # one JSONL file per session
├── <session-uuid>.jsonl
└── memory/                     # auto-memory (Ch 05)
    ├── MEMORY.md               # loaded every session
    ├── debugging.md            # loaded on demand
    └── ...
```

Each session is a single JSONL file — one JSON object per turn, plaintext, readable. This is the state that powers `/resume` (pick a session file) and `/rewind` (slice the JSONL at a turn boundary) and `/branch` (duplicate the JSONL, continue from the copy).

### The `.mcp.json` file

Project-scoped MCP server registrations live in `<repo>/.claude/.mcp.json`. Committed. Team members get your MCP server list automatically on clone. Covered in Ch 16.

## Why it matters

Four things become possible (or debuggable) once you understand the layout:

1. **Debugging weird behavior** — inspect the JSONL of a session to see exactly what turns happened. "Why did Claude call this tool?" has a literal answer in a text file.
2. **Portable setups** — commit `~/.claude/` to a personal dotfiles repo with a gitignore for `projects/` and secrets. Restore your environment on any machine (see cheatsheets in the primer repo).
3. **Programmatic access** — the Agent SDK [5] reads and writes the same files. The CLI and SDK aren't separate systems — they share state.
4. **Custom tooling** — you can `grep` across all your sessions for patterns, audit auto-memory, or build scripts over your Claude Code history.

## Debugging mental model

| Symptom | File to check |
|---|---|
| "Did Claude really run that command?" | `~/.claude/projects/<slug>/<session>.jsonl` — grep for the tool call |
| "What rules are loading?" | `/memory` in session, or manually walk CLAUDE.md hierarchy |
| "Auto-memory is writing weird stuff" | `~/.claude/projects/<slug>/memory/MEMORY.md` (and topic files) |
| "Skill isn't triggering" | `~/.claude/skills/<name>/SKILL.md` — check frontmatter, description (Ch 12) |
| "Permissions keep asking" | `~/.claude/settings.json` + `<repo>/.claude/settings.json` — merge inspection |

## Key takeaway

**Everything in Claude Code is a file.** CLAUDE.md, rules, skills, subagents, hooks, memory, sessions — all plaintext, all inspectable, all version-controllable. Treat the on-disk model as a first-class API. When the CLI seems mysterious, the state file will tell you what really happened.

## See Also

- [`01-mental-model.md`](01-mental-model.md) — Why this layout mirrors the harness
- [`05-auto-memory.md`](05-auto-memory.md) — The `memory/` subdirectory in depth
- [`09-session-lifecycle.md`](09-session-lifecycle.md) — Session JSONL semantics

## Sources

[1] Claude Code Docs — <https://code.claude.com/docs/en/>
[2] Claude Code Best Practices — <https://www.anthropic.com/engineering/claude-code-best-practices>
[5] Agent SDK overview — <https://code.claude.com/docs/en/agent-sdk/overview>
